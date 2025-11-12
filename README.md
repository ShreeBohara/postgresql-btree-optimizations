# PostgreSQL B-Tree Index Optimizations

Performance optimizations for PostgreSQL 17.4's B-tree index implementation, featuring linear search for small leaf pages and leaf-page prefetching for range scans.

## Overview

This project implements two micro-optimizations to PostgreSQL's B-tree index access methods:

1. **Linear Search Optimization**: Replaces binary search with linear scan for small leaf pages (≤4 items) to reduce overhead and improve cache locality
2. **Leaf-Page Prefetching**: Issues asynchronous prefetch requests for next sibling pages during range scans to overlap I/O with CPU processing

## Features

- ✅ Configurable via PostgreSQL GUC (Grand Unified Configuration) variables
- ✅ Backward compatible (disabled by default)
- ✅ Comprehensive benchmarking with 113 complex analytical queries
- ✅ Significant performance improvements for specific workloads

## Performance Results

### Linear Search Optimization
- **56.6%** of queries improved (64/113 queries)
- Average improvement: **80.41ms**
- Maximum improvement: **1,066ms**
- Best case: Query 6a.sql improved by **49.6%** (125ms → 63ms)

### Prefetch Optimization
- **31.0%** of queries improved (35/113 queries)
- Average improvement: **150.91ms** (when effective)
- Maximum improvement: **4,572ms**

### Combined Optimizations
- **37.2%** of queries improved (42/113 queries)
- Average improvement: **302.29ms**

## Installation

### Prerequisites
- PostgreSQL 17.4 source code
- Docker (for testing environment)
- Build tools: gcc, make, bison, flex

### Building PostgreSQL with Optimizations

1. Clone PostgreSQL 17.4:
```bash
wget https://ftp.postgresql.org/pub/source/v17.4/postgresql-17.4.tar.gz
tar xzf postgresql-17.4.tar.gz
cd postgresql-17.4
```

2. Apply the patch:
```bash
git apply /path/to/patches/btree_optimizations.patch
```

3. Configure and build:
```bash
CFLAGS=-O0 ./configure --prefix=/path/to/postgresql-17.4 --enable-debug
make -j8
make install
```

## Configuration

Three new GUC variables are available:

```sql
-- Enable linear search for small leaf pages
SET btree_binsrch_linear = on;

-- Set threshold for linear search (default: 4, range: 1-32)
SET btree_binsrch_linear_threshold = 4;

-- Enable leaf page prefetching
SET btree_leaf_prefetch = on;
```

## Benchmarking

We evaluated the optimizations using the IMDB/JOB (Join Order Benchmark) dataset with 113 complex analytical queries.

### Running Benchmarks

```bash
cd benchmarks
./benchmark_queries_linear.sh      # Test linear search optimization
./benchmark_queries_prefetch.sh    # Test prefetch optimization
./benchmark_queries_prefetch_linear.sh  # Test combined optimizations
```

Results are saved in `benchmark_results_*/` directories with detailed timing and comparison data.

## Technical Details

### Implementation Files Modified

- `src/include/access/nbtree.h` - External variable declarations
- `src/backend/access/nbtree/nbtutils.c` - Variable definitions
- `src/backend/access/nbtree/nbtsearch.c` - Core optimization logic
- `src/backend/utils/misc/guc_tables.c` - GUC system integration

### Linear Search Optimization

Implemented in `_bt_binsrch()` function. When enabled and a leaf page contains 2-4 items, performs sequential scan instead of binary search to:
- Eliminate loop overhead
- Improve CPU cache utilization
- Reduce branch misprediction penalties

### Prefetch Optimization

Implemented in `_bt_readpage()` function. During forward/backward range scans, asynchronously prefetches the next sibling leaf page using PostgreSQL's `PrefetchBuffer()` API to overlap I/O operations with CPU processing.

## Testing Environment

We provide a Docker-based testing environment:

```bash
docker build -t postgresql-btree-test -f docker/Dockerfile .
docker run --privileged -it --shm-size=32g \
  --cap-add=SYS_PTRACE --cap-add=CAP_SYS_ADMIN \
  postgresql-btree-test
```

## Results Analysis

The optimizations demonstrate workload-dependent performance characteristics:

- **Linear search** shows consistent benefits for queries accessing small leaf pages
- **Prefetch** provides significant gains for sequential range scans but can degrade performance for random access patterns
- **Combined optimizations** reveal complex interaction effects

See `docs/IMPLEMENTATION.md` for detailed technical analysis and discussion.

## Project Structure

```
postgresql-btree-optimizations/
├── patches/              # PostgreSQL patch files
├── benchmarks/           # Benchmark scripts and test queries
│   ├── job_queries/      # 113 IMDB/JOB benchmark queries
│   └── load-postgres/    # Database setup scripts
├── docker/               # Docker configuration
└── docs/                 # Documentation and analysis
```

## Contributing

This is a research/optimization project. Contributions, bug reports, and performance analysis are welcome.

## Authors

- **Shree Bohara** - Linear search optimization, GUC integration, benchmarking, performance analysis
- **Soumya** - Prefetch optimization, benchmark scripts, documentation, testing

See [CONTRIBUTORS.md](CONTRIBUTORS.md) for detailed contribution breakdown.

## License

This project is provided as-is for educational and research purposes. PostgreSQL is licensed under the PostgreSQL License (similar to BSD/MIT).

## Acknowledgments

- PostgreSQL Global Development Group for the excellent database system
- IMDB/JOB benchmark creators for the comprehensive test suite

