# Metabase with DuckDB Support

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A ready-to-use Docker image that integrates [Metabase](https://www.metabase.com/) with the [DuckDB](https://duckdb.org/) driver, allowing you to connect to and visualize DuckDB databases.

This project packages Metabase v0.52.9 with the community-maintained DuckDB driver v0.3.0.

## 🚀 Quick Start

### Option 1: Use the GitHub Container Registry image

```bash
docker pull ghcr.io/carrilloapps/metabase-duckdb:latest

# Run standalone container
docker run -d -p 3000:3000 ghcr.io/carrilloapps/metabase-duckdb:latest

# Or with PostgreSQL for persistence using docker-compose
curl -O https://raw.githubusercontent.com/carrilloapps/metabase-duckdb/main/docker-compose.yaml
docker-compose up -d
```

### Option 2: Build locally

```bash
git clone https://github.com/carrilloapps/metabase-duckdb.git
cd metabase-duckdb
docker-compose up -d
```

Once running, access Metabase at http://localhost:3000

## 📋 Features

- **Ready-to-use Metabase installation**: Pre-configured with Metabase v0.52.9
- **DuckDB support**: Includes the DuckDB driver v0.3.0
- **PostgreSQL persistence**: Use the included PostgreSQL database for storing Metabase metadata
- **Docker Compose setup**: Simple deployment with Docker Compose

## 🔧 Configuration

### Environment Variables

The following environment variables can be configured:

| Variable | Description | Default |
|----------|-------------|---------|
| `SERVICE_USER_POSTGRESQL` | PostgreSQL username | `metabase_user` |
| `SERVICE_PASSWORD_POSTGRESQL` | PostgreSQL password | `tu_contraseña_segura` |
| `POSTGRESQL_DATABASE` | PostgreSQL database name | `metabase` |

For convenience, an `.env.example` file is provided. Copy it to `.env` and customize as needed:

```bash
cp .env.example .env
# Edit .env with your preferred text editor
```

### Connecting to DuckDB

1. Log in to Metabase
2. Go to Settings > Admin > Databases > Add database
3. Select "DuckDB" from the dropdown
4. Enter your DuckDB file path (for mounted volumes) or use `:memory:` for an in-memory database
5. Test the connection and save

## 🏗️ Project Structure

```
metabase-duckdb/
├── Dockerfile             # Defines the Metabase image with DuckDB driver
├── docker-compose.yaml    # Orchestrates Metabase and PostgreSQL containers
├── README.md              # This documentation
└── docs/                  # Additional documentation
```

## 📚 Documentation

Additional documentation can be found in the [docs](./docs) directory:

- [DuckDB Integration Guide](./docs/DUCKDB_GUIDE.md)
- [Example DuckDB Queries](./docs/EXAMPLE_QUERIES.md)
- [Docker Volumes Guide](./docs/VOLUMES.md)
- [Contributing Guide](./docs/CONTRIBUTING.md)
- [Changelog](./docs/CHANGELOG.md)
- [Code of Conduct](./docs/CODE_OF_CONDUCT.md)
- [GitHub Container Registry Usage](./docs/GHCR_USAGE.md)
- [Troubleshooting Guide](./docs/TROUBLESHOOTING.md)
- [Development Guide](./docs/DEVELOPMENT.md)
- [Security Best Practices](./docs/SECURITY.md)

## ⚖️ License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

## 📜 Legal

This project is not affiliated with Metabase or DuckDB. Metabase and DuckDB are used in accordance with their respective licenses:

- Metabase is licensed under the AGPL
- The DuckDB driver is maintained by the community and licensed under its own terms

## 🙏 Acknowledgements

- [Metabase](https://www.metabase.com/) for their excellent open-source BI tool
- [DuckDB](https://duckdb.org/) for the innovative analytical database
- [MotherDuck](https://github.com/motherduckdb/metabase_duckdb_driver) for maintaining the DuckDB driver for Metabase