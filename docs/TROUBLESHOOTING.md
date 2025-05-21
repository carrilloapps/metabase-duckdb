# Troubleshooting Guide

This document provides solutions for common issues you might encounter when using the Metabase-DuckDB Docker image.

## Container Won't Start

### Symptoms
- Container exits immediately after starting
- You see "Exited (1)" status in `docker ps -a`

### Possible Causes and Solutions

1. **Insufficient Memory**
   
   Metabase requires at least 1GB of RAM to run properly.
   
   **Solution**: Ensure your Docker host has enough free memory. You can limit Metabase's memory usage using the `-m` option:
   ```bash
   docker run -d -p 3000:3000 -m 1G ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

2. **Port Conflict**
   
   Port 3000 might already be in use on your system.
   
   **Solution**: Map to a different port:
   ```bash
   docker run -d -p 8080:3000 ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

3. **Permission Issues**
   
   **Solution**: Check the container logs for permission errors:
   ```bash
   docker logs <container_id>
   ```

## Connection to DuckDB Fails

### Symptoms
- Error when trying to connect to a DuckDB database
- "Connection failed" message in Metabase UI

### Possible Causes and Solutions

1. **Incorrect File Path**
   
   **Solution**: Ensure you're providing the correct path to the DuckDB file and that it's accessible to the container. For files on the host, you need to mount them:
   ```bash
   docker run -d -p 3000:3000 -v /path/to/your/data:/data ghcr.io/carrilloapps/metabase-duckdb:latest
   ```
   
   Then in Metabase, use `/data/your-file.duckdb` as the connection path.

2. **Using :memory: Database**
   
   If you're using the `:memory:` option, remember that the data is not persistent and will be lost when the container restarts.

3. **DuckDB Driver Issues**
   
   **Solution**: Check if the DuckDB driver is properly loaded in Metabase:
   1. Go to Admin > Troubleshooting > Logs
   2. Look for any errors related to the DuckDB driver
   3. If necessary, restart the container

## PostgreSQL Connection Issues

### Symptoms
- Metabase fails to connect to PostgreSQL
- Error messages about database connection

### Possible Causes and Solutions

1. **Incorrect Configuration**
   
   **Solution**: Verify your PostgreSQL environment variables:
   ```bash
   docker run -d -p 3000:3000 \
     -e MB_DB_TYPE=postgres \
     -e MB_DB_HOST=postgresql \
     -e MB_DB_PORT=5432 \
     -e MB_DB_DBNAME=metabase \
     -e MB_DB_USER=metabase_user \
     -e MB_DB_PASS=your_secure_password \
     ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

2. **Network Issues**
   
   **Solution**: Ensure the PostgreSQL container is reachable from the Metabase container. If using Docker Compose, make sure they're on the same network.

## Performance Issues

### Symptoms
- Slow response times
- Queries take too long to execute

### Possible Causes and Solutions

1. **Insufficient Resources**
   
   **Solution**: Allocate more resources to the container:
   ```bash
   docker run -d -p 3000:3000 --memory=2g --cpus=2 ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

2. **Large Datasets**
   
   **Solution**: For large datasets, consider:
   - Optimizing your queries
   - Creating appropriate indexes in your DuckDB database
   - Increasing the JVM heap size with `-e MB_JAVA_OPTS="-Xmx2g"`

## Container Crashes After Running for a While

### Symptoms
- Container stops unexpectedly after running for some time

### Possible Causes and Solutions

1. **Out of Memory**
   
   **Solution**: Increase the memory limit and add swap:
   ```bash
   docker run -d -p 3000:3000 --memory=2g --memory-swap=4g ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

2. **Java Heap Issues**
   
   **Solution**: Adjust Java heap settings:
   ```bash
   docker run -d -p 3000:3000 -e MB_JAVA_OPTS="-Xmx1g" ghcr.io/carrilloapps/metabase-duckdb:latest
   ```

## Still Having Issues?

If you've tried these solutions and are still experiencing problems:

1. Check the full container logs:
   ```bash
   docker logs --tail=100 <container_id>
   ```

2. Create an issue on our GitHub repository with:
   - Detailed description of the problem
   - Steps to reproduce
   - Logs and error messages
   - Your environment details (Docker version, host OS, etc.)

3. For urgent issues, consider using the official Metabase or DuckDB community resources.
