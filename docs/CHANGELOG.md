# Changelog

All notable changes to the Metabase-DuckDB project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2025-05-21

### Added
- Initial release of Metabase-DuckDB Docker image
- Metabase v0.52.9 with DuckDB driver v0.3.0
- Docker Compose configuration with PostgreSQL for metadata storage
- Basic documentation including README, CONTRIBUTING, CODE_OF_CONDUCT, and LICENSE
- GitHub Actions workflows for CI/CD and publishing to GHCR

### Changed
- Consolidated GitHub Actions workflows (`docker-build-push.yml` and `publish.yml`) into a single optimized workflow
- Improved Docker image tagging strategy with support for both `main` and `release` branches
- Updated Docker build actions to use the latest versions

### Security
- Set up secure default configuration
- Configured healthchecks for service monitoring
