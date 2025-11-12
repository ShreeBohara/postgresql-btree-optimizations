# Implementation Details

## Overview

This document provides detailed technical information about the B-tree optimizations implemented in PostgreSQL 17.4.

## Architecture

### B-tree Structure in PostgreSQL

PostgreSQL B-trees follow the Lehman-Yao design, consisting of:
- **Internal nodes**: Contain separator keys and child pointers
- **Leaf nodes**: Contain actual data and tuple identifiers (TIDs)
- **Sibling pointers**: Leaf pages are linked via `btpo_next` and `btpo_prev` for efficient range scans

A typical index scan involves:
1. Locating the first matching leaf (`_bt_first`)
2. Repeatedly advancing across items and sibling leaves (`_bt_next`)
3. Reading and interpreting page contents (`_bt_readpage`)
4. Pinpointing item positions via binary search (`_bt_binsrch`)

## Optimization 1: Linear Search for Small Leaf Pages

### Motivation

When a leaf page contains very few items (e.g., ≤4), binary search overhead can exceed its benefits:
- Loop overhead for binary search iterations
- Branch misprediction penalties
- Poor cache locality from non-sequential access

### Implementation

**Location**: `src/backend/access/nbtree/nbtsearch.c` in `_bt_binsrch()` function

**Algorithm**:
1. Check if linear search optimization is enabled (`btree_binsrch_linear`)
2. Verify page is a leaf page (`P_ISLEAF(opaque)`)
3. Count items on page (`high - low + 1`)
4. If item count is between 2 and threshold, perform linear scan
5. Otherwise, fall back to standard binary search

**Key Features**:
- Configurable threshold (default: 4 items)
- Handles both `nextkey` and non-`nextkey` scan types
- Maintains same return semantics as binary search
- Zero overhead when disabled

### Code Structure

```c
if (btree_binsrch_linear && P_ISLEAF(opaque)) {
    int num_items = high - low + 1;
    
    if (num_items >= 2 && num_items <= btree_binsrch_linear_threshold) {
        /* Perform linear search */
        for (offset = low; offset <= high; offset++) {
            result = _bt_compare(rel, key, page, offset);
            /* Handle nextkey and non-nextkey cases */
        }
    }
}
/* Continue with normal binary search */
```

### Performance Characteristics

- **Best case**: Small leaf pages with 2-4 items
- **Worst case**: Large leaf pages (falls back to binary search)
- **Overhead**: Minimal - single conditional check when disabled

## Optimization 2: Leaf-Page Prefetching

### Motivation

Range scans often access multiple consecutive leaf pages. By prefetching the next page while processing the current one, we can overlap I/O with CPU processing, reducing overall query latency.

### Implementation

**Location**: `src/backend/access/nbtree/nbtsearch.c` in `_bt_readpage()` function

**Algorithm**:
1. Check if prefetch optimization is enabled (`btree_leaf_prefetch`)
2. Verify page is a leaf page (`P_ISLEAF(opaque)`)
3. Determine scan direction (forward/backward)
4. Check if scan will continue (`moreRight`/`moreLeft`)
5. Extract next page block number from sibling pointer
6. Issue asynchronous prefetch request using `PrefetchBuffer()`

**Key Features**:
- Supports both forward and backward scans
- Only prefetches when scan will continue
- Uses PostgreSQL's built-in prefetch API
- Non-blocking - doesn't affect current page processing

### Code Structure

```c
if (btree_leaf_prefetch && P_ISLEAF(opaque)) {
    BlockNumber nextblk = InvalidBlockNumber;
    
    if (ScanDirectionIsForward(dir) && so->currPos.moreRight)
        nextblk = opaque->btpo_next;
    else if (ScanDirectionIsBackward(dir) && so->currPos.moreLeft)
        nextblk = opaque->btpo_prev;
    
    if (BlockNumberIsValid(nextblk))
        PrefetchBuffer(scan->indexRelation, MAIN_FORKNUM, nextblk);
}
```

### Performance Characteristics

- **Best case**: Sequential range scans across many pages
- **Worst case**: Random access patterns or early scan termination
- **Trade-off**: May cause cache pollution if prefetched page isn't accessed

## GUC System Integration

### Configuration Variables

Three new GUC variables were added:

1. **btree_binsrch_linear** (boolean, default: off)
   - Enables linear search for small leaf pages
   - Category: DEVELOPER_OPTIONS
   - Scope: PGC_USERSET (can be changed per session)

2. **btree_binsrch_linear_threshold** (integer, default: 4, range: 1-32)
   - Maximum number of items for linear search activation
   - Category: DEVELOPER_OPTIONS
   - Scope: PGC_USERSET

3. **btree_leaf_prefetch** (boolean, default: off)
   - Enables leaf page prefetching
   - Category: DEVELOPER_OPTIONS
   - Scope: PGC_USERSET

### Files Modified

1. **src/include/access/nbtree.h**
   - External variable declarations

2. **src/backend/access/nbtree/nbtutils.c**
   - Variable definitions with default values

3. **src/backend/access/nbtree/nbtsearch.c**
   - Core optimization logic in `_bt_binsrch()` and `_bt_readpage()`

4. **src/backend/utils/misc/guc_tables.c**
   - GUC registration and header inclusion

## Performance Analysis

### Linear Search Results

- **56.6%** of queries improved (64/113)
- Average improvement: **80.41ms**
- Maximum improvement: **1,066ms**
- Notable improvements:
  - Query 6a.sql: 125ms → 63ms (49.6% improvement)
  - Query 28c.sql: 964ms → 741ms (23.1% improvement)
  - Query 25c.sql: 10,656ms → 9,689ms (9.1% improvement)

### Prefetch Results

- **31.0%** of queries improved (35/113)
- Average improvement: **150.91ms** (when effective)
- Maximum improvement: **4,572ms**
- Notable improvements:
  - Query 25c.sql: 10,225ms → 9,755ms (470ms improvement)
- Notable regressions:
  - Query 6d.sql: 9,246ms → 13,818ms (cache pollution effect)

### Combined Optimization Results

- **37.2%** of queries improved (42/113)
- Average improvement: **302.29ms**
- Shows complex interaction effects between optimizations

## Design Decisions

### Why Threshold of 4?

The default threshold of 4 items was chosen based on:
- Binary search overhead becomes significant for small arrays
- Linear search is faster for ≤4 items on modern CPUs
- Larger thresholds risk performance degradation

### Why Prefetch Only Next Page?

- Prefetching multiple pages ahead increases cache pollution risk
- Single-page prefetch provides good I/O overlap without excessive overhead
- Can be extended to multi-page prefetching in future work

### Why Disabled by Default?

- Optimizations are workload-dependent
- Some queries may perform worse with optimizations enabled
- Allows users to enable based on their specific workload characteristics

## Future Work

Potential improvements:
1. Adaptive threshold adjustment based on page access patterns
2. Smarter prefetch decisions using query plan information
3. Integration with PostgreSQL's cost-based optimizer
4. Multi-page prefetching for long sequential scans
5. Prefetch cancellation for early scan termination

## References

- PostgreSQL Documentation: https://www.postgresql.org/docs/17/
- Lehman-Yao B-tree Design: Efficient locking for concurrent operations on B-trees (TODS, 1981)
- IMDB/JOB Benchmark: How good are query optimizers, really? (VLDB, 2015)

