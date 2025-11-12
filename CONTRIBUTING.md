# Contributing

Thank you for your interest in contributing to this project! This document provides guidelines and instructions for contributing.

## Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/YOUR_USERNAME/postgresql-btree-optimizations.git`
3. Create a branch: `git checkout -b feature/your-feature-name`

## Development Setup

1. Build PostgreSQL 17.4 with the patch applied (see README.md)
2. Set up the testing environment using Docker
3. Run benchmarks to verify your changes

## Making Changes

### Code Style

- Follow PostgreSQL coding conventions
- Use consistent indentation (tabs for PostgreSQL code)
- Add comments for complex logic
- Keep functions focused and modular

### Testing

- Test your changes with the provided benchmark suite
- Verify backward compatibility (optimizations disabled)
- Test edge cases (empty pages, single-item pages, etc.)
- Document any performance regressions

### Documentation

- Update README.md if adding new features
- Add implementation details to docs/IMPLEMENTATION.md
- Update CHANGELOG.md with your changes

## Submitting Changes

1. Ensure all tests pass
2. Update documentation as needed
3. Write a clear commit message
4. Push to your fork: `git push origin feature/your-feature-name`
5. Create a pull request with:
   - Description of changes
   - Performance impact (if applicable)
   - Test results

## Areas for Contribution

- Performance improvements
- Additional optimizations
- Bug fixes
- Documentation improvements
- Benchmark enhancements
- Analysis of optimization effectiveness

## Core Team

- **Shree Bohara** - Co-lead Developer (Linear search optimization, GUC integration, benchmarking)
- **Soumya** - Co-lead Developer (Prefetch optimization, benchmark scripts, documentation)

## Questions?

Feel free to open an issue for questions or discussions.

