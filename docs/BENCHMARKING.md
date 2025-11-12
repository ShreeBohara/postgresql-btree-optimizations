# Benchmarking Guide

## Overview

This document describes how to run benchmarks and evaluate the B-tree optimizations.

## Benchmark Dataset

We use the **IMDB/JOB (Join Order Benchmark)** dataset, which consists of:
- Real-world IMDB database schema
- 113 complex analytical queries
- Designed to stress-test query optimization and execution

## Benchmark Scripts

Three benchmark scripts are provided:

1. **benchmark_queries_linear.sh** - Tests linear search optimization
2. **benchmark_queries_prefetch.sh** - Tests prefetch optimization
3. **benchmark_queries_prefetch_linear.sh** - Tests combined optimizations

## Methodology

### Dual-Loop Design

Each benchmark uses a dual-loop A/B testing methodology to control for cache effects:

- **Loop 1**: Optimization ON first, then OFF
- **Loop 2**: Optimization OFF first, then ON
- **Final Results**: Average of both loops

This design helps account for:
- Buffer cache warm-up effects
- System load variations
- Measurement noise

### Measurement

- **Precision**: Millisecond-level timing (`date +%s%3N`)
- **Timeout**: 300 seconds per query
- **PostgreSQL Timing**: Uses `\timing` command for query execution time
- **Wall-clock Time**: Captures total script execution time

## Running Benchmarks

### Prerequisites

1. PostgreSQL 17.4 with optimizations applied
2. IMDB database loaded (see `benchmarks/load-postgres/`)
3. Database connection configured in benchmark scripts

### Configuration

Edit the benchmark scripts to set:
```bash
DB_NAME="imdbload"
DB_USER="your_username"
DB_HOST="localhost"
DB_PORT="5432"
```

### Execution

```bash
cd benchmarks

# Test linear search optimization
./benchmark_queries_linear.sh

# Test prefetch optimization
./benchmark_queries_prefetch.sh

# Test combined optimizations
./benchmark_queries_prefetch_linear.sh
```

### Output

Each benchmark creates a results directory:
- `benchmark_results_linear/` - Linear search results
- `benchmark_results_prefetch/` - Prefetch results
- `benchmark_results_prefetch_linear/` - Combined results

Each directory contains:
- `query_benchmark_results.txt` - Detailed log file
- `query_timing.csv` - Per-query timing data
- `performance_comparison.csv` - Comparison statistics
- Individual query result files (`*.result`, `*.error`)

## Interpreting Results

### Performance Comparison CSV

The `performance_comparison.csv` file contains:
- `query_file` - SQL query filename
- `off_duration_ms` - Average execution time with optimization OFF
- `on_duration_ms` - Average execution time with optimization ON
- `performance_improvement_ms` - Difference (positive = faster with ON)
- `improvement_percent` - Percentage improvement

### Summary Statistics

Each benchmark generates summary statistics:
- Total queries tested
- Number of queries improved/degraded
- Average improvement when faster
- Average degradation when slower
- Maximum improvement observed

### Example Output

```
Total queries tested: 113
btree_binsrch_linear = on faster: 64 (56.6%)
btree_binsrch_linear = off faster: 47 (41.6%)
Average improvement when on is faster: 80.41ms
Average improvement when off is faster: 95.32ms
Maximum improvement observed: 1066.00ms
```

## Analyzing Results

### Identifying Beneficial Queries

Look for queries with:
- Large positive `performance_improvement_ms`
- High `improvement_percent` (>10%)
- Consistent improvement across both loops

### Understanding Regressions

Queries that perform worse may indicate:
- Cache pollution (prefetch optimization)
- Threshold too high (linear search)
- Query characteristics not suited for optimization

### Workload Analysis

Analyze patterns:
- Which query types benefit most?
- Are improvements consistent or variable?
- Do optimizations interact positively or negatively?

## Troubleshooting

### Common Issues

1. **Connection errors**: Check database credentials and PostgreSQL is running
2. **Timeout errors**: Increase timeout or optimize slow queries
3. **Permission errors**: Ensure script has execute permissions (`chmod +x`)
4. **Missing queries**: Verify `job_queries/` directory contains all SQL files

### Debugging

Enable verbose output by modifying scripts:
```bash
set -x  # Enable debug mode
```

Check individual query results:
```bash
cat benchmark_results_linear/1a.sql_on_loop1.result
cat benchmark_results_linear/1a.sql_on_loop1.error
```

## Best Practices

1. **Run benchmarks multiple times** to account for system variability
2. **Clear buffer cache** between major test runs (if needed)
3. **Monitor system resources** during benchmarking
4. **Document environment** (PostgreSQL version, hardware, OS)
5. **Compare against baseline** (unmodified PostgreSQL)

## Advanced Usage

### Custom Threshold Testing

Modify threshold and re-run:
```sql
SET btree_binsrch_linear_threshold = 8;
```

### Selective Query Testing

Modify scripts to test specific queries:
```bash
SQL_FILES="job_queries/6a.sql job_queries/25c.sql"
```

### Performance Profiling

Use PostgreSQL's profiling tools:
```sql
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT ...;
```

