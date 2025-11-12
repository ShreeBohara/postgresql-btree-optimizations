# Quick Start Guide

Get up and running with PostgreSQL B-tree optimizations in minutes.

## Prerequisites

- Linux/macOS system
- Docker installed (optional, for testing environment)
- PostgreSQL 17.4 source code
- Build tools: gcc, make, bison, flex

## Step 1: Get PostgreSQL Source

```bash
cd ~
wget https://ftp.postgresql.org/pub/source/v17.4/postgresql-17.4.tar.gz
tar xzf postgresql-17.4.tar.gz
cd postgresql-17.4
```

## Step 2: Apply the Patch

```bash
git apply /path/to/postgresql-btree-optimizations/patches/btree_optimizations.patch
```

Or if using git:
```bash
git init
git add .
git commit -m "Initial PostgreSQL 17.4"
git apply /path/to/postgresql-btree-optimizations/patches/btree_optimizations.patch
```

## Step 3: Build PostgreSQL

```bash
CFLAGS=-O0 ./configure --prefix=$HOME/postgresql-17.4 --enable-debug
make -j8  # Adjust -j based on your CPU cores
make install

# Add to PATH
export PATH=$HOME/postgresql-17.4/bin:$PATH
```

## Step 4: Initialize Database

```bash
mkdir -p ~/databases
initdb -D ~/databases
```

## Step 5: Start PostgreSQL

```bash
pg_ctl -D ~/databases -l ~/databases/logfile start
```

## Step 6: Enable Optimizations

Connect to PostgreSQL:
```bash
psql -d postgres
```

Enable optimizations:
```sql
SET btree_binsrch_linear = on;
SET btree_binsrch_linear_threshold = 4;
SET btree_leaf_prefetch = on;
```

## Step 7: Test with a Query

```sql
-- Create a test table and index
CREATE TABLE test_table (id INT, data TEXT);
CREATE INDEX idx_test ON test_table(id);

-- Insert test data
INSERT INTO test_table SELECT generate_series(1, 10000), 'data';

-- Test query
EXPLAIN ANALYZE SELECT * FROM test_table WHERE id BETWEEN 1000 AND 2000;
```

## Using Docker (Alternative)

For a complete testing environment:

```bash
cd postgresql-btree-optimizations
docker build -t postgresql-btree-test -f docker/Dockerfile .

docker run --privileged -it --shm-size=32g \
  --cap-add=SYS_PTRACE --cap-add=CAP_SYS_ADMIN \
  -v $(pwd):/workspace \
  postgresql-btree-test
```

Inside the container:
```bash
cd /workspace
# Follow steps 1-7 above
```

## Running Benchmarks

See `docs/BENCHMARKING.md` for detailed benchmarking instructions.

Quick benchmark:
```bash
cd benchmarks
export DB_USER=your_username
export DB_NAME=imdbload
./benchmark_queries_linear.sh
```

## Troubleshooting

### Build Errors
- Ensure all dependencies are installed
- Check PostgreSQL build requirements: https://www.postgresql.org/docs/17/install-requirements.html

### Patch Application Errors
- Verify you're using PostgreSQL 17.4 exactly
- Check patch file path is correct
- Ensure source code is clean (no uncommitted changes)

### Database Connection Issues
- Verify PostgreSQL is running: `pg_ctl status -D ~/databases`
- Check port conflicts: `netstat -an | grep 5432`
- Review log file: `cat ~/databases/logfile`

## Next Steps

- Read `docs/IMPLEMENTATION.md` for technical details
- Review `docs/BENCHMARKING.md` for performance evaluation
- Check `README.md` for full documentation

