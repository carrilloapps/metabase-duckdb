# Contributing to Metabase-DuckDB

Thank you for your interest in contributing to Metabase-DuckDB! This document provides guidelines and instructions for contributing to this project.

## Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](./CODE_OF_CONDUCT.md).

## How to Contribute

### Reporting Issues

If you encounter any issues or have feature requests, please submit them through GitHub Issues. When creating an issue, please:

1. Check if a similar issue already exists
2. Use a clear and descriptive title
3. Provide detailed steps to reproduce the issue
4. Include information about your environment (OS, Docker version, etc.)
5. Add relevant screenshots or logs if possible

### Pull Requests

We welcome contributions through pull requests. To submit a pull request:

1. Fork the repository
2. Create a new branch for your changes (`git checkout -b feature/your-feature-name`)
3. Make your changes
4. Test your changes thoroughly
5. Commit your changes with clear, descriptive messages
6. Push your changes to your fork
7. Submit a pull request to our main repository

#### Pull Request Guidelines

- Follow the existing code style and conventions
- Keep changes focused and related to a single issue
- Document new code and update existing documentation if necessary
- Include tests for new functionality
- Ensure all tests pass
- Keep PRs reasonably sized to make review easier

### Development Environment Setup

To set up a local development environment:

1. Clone the repository:
   ```bash
   git clone https://github.com/carrilloapps/metabase-duckdb.git
   cd metabase-duckdb
   ```

2. Build and run the containers:
   ```bash
   docker-compose up -d --build
   ```

3. Access Metabase at http://localhost:3000

### Pull Request Process

1. Update the README.md or other documentation with details of changes if needed
2. Your PR will be reviewed by maintainers, who may request changes
3. Once approved, your PR will be merged by a maintainer

## Documentation

Good documentation is crucial for this project. If you make changes that affect the user experience, please update the relevant documentation.

## License

By contributing to this project, you agree that your contributions will be licensed under the project's [MIT License](../LICENSE).
