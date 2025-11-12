# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2025

**Developed by:** Shree Bohara & Soumya

### Added
- Linear search optimization for small B-tree leaf pages (≤4 items)
- Leaf-page prefetching optimization for range scans
- Three configurable GUC variables:
  - `btree_binsrch_linear` (boolean)
  - `btree_binsrch_linear_threshold` (integer, 1-32)
  - `btree_leaf_prefetch` (boolean)
- Comprehensive benchmarking suite with 113 IMDB/JOB queries
- Docker-based testing environment
- Complete documentation and implementation guides

### Performance
- Linear search: 56.6% query improvement rate, up to 1,066ms gains
- Prefetch: 31.0% query improvement rate, up to 4,572ms gains
- Combined: 37.2% query improvement rate, average 302ms gains

### Technical Details
- Modified PostgreSQL 17.4 core B-tree implementation
- Integrated with PostgreSQL GUC system
- Backward compatible (disabled by default)
- Zero overhead when optimizations are disabled

