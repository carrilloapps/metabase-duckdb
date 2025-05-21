# Using the GitHub Container Registry Image

This document explains how to pull and run the Metabase-DuckDB image from GitHub Container Registry (GHCR).

## Prerequisites

- Docker installed on your system
- Basic knowledge of Docker commands

## Pulling the Image

To pull the latest version of the Metabase-DuckDB image:

```bash
docker pull ghcr.io/yourusername/metabase-duckdb:latest
```

You can also pull a specific version:

```bash
docker pull ghcr.io/yourusername/metabase-duckdb:v1.0.0
```

## Running the Container

### Basic Usage

To run the container with default settings:

```bash
docker run -d -p 3000:3000 --name metabase-duckdb ghcr.io/yourusername/metabase-duckdb:latest
```

This will start Metabase on port 3000. You can access it by navigating to `http://localhost:3000` in your web browser.

### With Persistent Storage

For production use, you should run the container with persistent storage:

```bash
# Create a volume for Metabase data
docker volume create metabase-data

# Run with the volume mounted
docker run -d -p 3000:3000 --name metabase-duckdb \
  -v metabase-data:/metabase-data \
  ghcr.io/yourusername/metabase-duckdb:latest
```

### With Docker Compose

For a more complete setup with PostgreSQL for metadata storage, you can use Docker Compose:

1. Create a `docker-compose.yml` file:

```yaml
services:
  metabase:
    image: ghcr.io/yourusername/metabase-duckdb:latest
    platform: linux/amd64
    ports:
      - '3000:3000'
    volumes:
      - '/dev/urandom:/dev/random:ro'
    environment:
      - MB_DB_TYPE=postgres
      - MB_DB_HOST=postgresql
      - MB_DB_PORT=5432
      - 'MB_DB_DBNAME=${POSTGRESQL_DATABASE:-metabase}'
      - MB_DB_USER=${SERVICE_USER_POSTGRESQL:-metabase_user}
      - MB_DB_PASS=${SERVICE_PASSWORD_POSTGRESQL:-your_secure_password}
    mem_limit: 1g
    mem_reservation: 512m
    healthcheck:
      test: 'curl --fail -I http://127.0.0.1:3000/api/health || exit 1'
      interval: 5s
      timeout: 20s
      retries: 10
    depends_on:
      postgresql:
        condition: service_healthy

  postgresql:
    image: 'postgres:16-alpine'
    platform: linux/amd64
    ports:
      - '5432:5432'
    volumes:
      - 'postgresql-data:/var/lib/postgresql/data'
    environment:
      - 'POSTGRES_USER=${SERVICE_USER_POSTGRESQL:-metabase_user}'
      - 'POSTGRES_PASSWORD=${SERVICE_PASSWORD_POSTGRESQL:-your_secure_password}'
      - 'POSTGRES_DB=${POSTGRESQL_DATABASE:-metabase}'
    healthcheck:
      test:
        - CMD-SHELL
        - 'pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}'
      interval: 5s
      timeout: 20s
      retries: 10

volumes:
  postgresql-data:
    name: metabase-postgresql-data
```

2. Start the services:

```bash
docker-compose up -d
```

## Environment Variables

You can customize the container behavior using environment variables:

```bash
docker run -d -p 3000:3000 --name metabase-duckdb \
  -e MB_DB_TYPE=postgres \
  -e MB_DB_HOST=your-postgres-host \
  -e MB_DB_PORT=5432 \
  -e MB_DB_DBNAME=metabase \
  -e MB_DB_USER=metabase_user \
  -e MB_DB_PASS=your_secure_password \
  ghcr.io/yourusername/metabase-duckdb:latest
```

## Troubleshooting

If you encounter issues:

1. Check the container logs:
   ```bash
   docker logs metabase-duckdb
   ```

2. Verify that the container is running:
   ```bash
   docker ps | grep metabase-duckdb
   ```

3. Check if the port is correctly mapped:
   ```bash
   docker port metabase-duckdb
   ```

## Security Considerations

- For production use, always change default passwords
- Consider using Docker secrets for sensitive information
- Set up proper network configurations to limit access to your containers
