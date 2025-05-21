# Development Guide

This guide provides instructions for developers who want to contribute to the Metabase-DuckDB project by modifying or extending the Docker image.

## Setting Up the Development Environment

### Prerequisites

- Docker and Docker Compose installed
- Git for version control
- A code editor (VS Code, IntelliJ, etc.)
- Basic knowledge of Docker, Metabase, and DuckDB

### Getting the Source Code

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/metabase-duckdb.git
   cd metabase-duckdb
   ```

2. Create a new branch for your changes:
   ```bash
   git checkout -b feature/your-feature-name
   ```

## Making Changes

### Modifying the Dockerfile

The Dockerfile is the core of this project. It defines how the Metabase image is built with DuckDB support.

Key aspects of the Dockerfile:
- Base image: `openjdk:19-buster`
- Metabase version: currently v0.52.9
- DuckDB driver version: currently v0.3.0

If you need to update these versions, modify the corresponding lines:

```dockerfile
ADD https://downloads.metabase.com/v0.52.9/metabase.jar /home
ADD https://github.com/motherduckdb/metabase_duckdb_driver/releases/download/0.3.0/duckdb.metabase-driver.jar /home/plugins/
```

### Testing Your Changes

1. Build the modified image:
   ```bash
   docker-compose build
   ```

2. Run the container:
   ```bash
   docker-compose up -d
   ```

3. Test Metabase by accessing http://localhost:3000

4. Check the logs for any errors:
   ```bash
   docker-compose logs -f metabase
   ```

### Adding Additional Drivers or Features

To add additional database drivers to the image:

1. Modify the Dockerfile to include the new driver:
   ```dockerfile
   ADD https://path/to/driver.jar /home/plugins/
   RUN chmod 744 /home/plugins/driver.jar
   ```

2. Update the documentation to reflect the added capability

## Advanced Development

### Customizing Metabase Properties

You can customize Metabase by adding environment variables to the container in `docker-compose.yaml`:

```yaml
environment:
  - MB_CUSTOM_PROPERTY=value
```

For a full list of supported environment variables, refer to the [Metabase documentation](https://www.metabase.com/docs/latest/installation-and-operation/environment-variables.html).

### Creating a Multi-Architecture Build

For broader compatibility, you can build the image for multiple architectures using Docker Buildx:

```bash
docker buildx create --name mybuilder --use
docker buildx build --platform linux/amd64,linux/arm64 -t yourusername/metabase-duckdb:latest --push .
```

## Debugging

### Debugging Metabase

To debug Metabase issues:

1. Enable TRACE or DEBUG level logging:
   ```yaml
   environment:
     - MB_LOGGING_LEVEL=DEBUG
   ```

2. Check the logs:
   ```bash
   docker-compose logs -f metabase
   ```

### Debugging Driver Issues

To troubleshoot DuckDB driver issues:

1. Verify the driver is correctly loaded by checking the Metabase logs during startup
2. Check for driver-specific logs in the Metabase Admin > Troubleshooting section
3. If necessary, manually check the plugins directory contents:
   ```bash
   docker exec -it metabase-duckdb ls -la /home/plugins/
   ```

## Submitting Changes

After making and testing your changes:

1. Commit your changes with a clear message:
   ```bash
   git add .
   git commit -m "Add feature X" or "Fix issue Y"
   ```

2. Push to your forked repository:
   ```bash
   git push origin feature/your-feature-name
   ```

3. Create a Pull Request following the contribution guidelines in [CONTRIBUTING.md](./CONTRIBUTING.md)

## Best Practices

- Keep changes focused and minimal
- Follow the existing code style
- Update documentation for any changes
- Write clear commit messages
- Test thoroughly before submitting
- Consider backward compatibility
- Include version updates in the changelog
