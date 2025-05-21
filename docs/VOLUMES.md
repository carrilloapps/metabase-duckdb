# Using Docker Volumes with Metabase-DuckDB

This guide explains how to use Docker volumes effectively with the Metabase-DuckDB container for data persistence and working with DuckDB databases.

## Types of Data to Persist

When running Metabase-DuckDB, you may want to persist several types of data:

1. **Metabase application data** - Normally handled by the PostgreSQL container
2. **DuckDB database files** - The actual DuckDB database files
3. **Data files for import** - CSV, Parquet, or other files for import into DuckDB

## Setting Up Volumes

### PostgreSQL Data (Recommended Approach)

The recommended approach for Metabase application data is to use the PostgreSQL container as configured in the docker-compose.yaml:

```yaml
volumes:
  postgresql-data:
    name: metabase-postgresql-data
```

This creates a named volume for PostgreSQL data, ensuring that Metabase settings, questions, dashboards, and user data persist between container restarts.

### DuckDB Database Files

To work with DuckDB database files, you should create a separate volume:

```yaml
services:
  metabase:
    # ... other configuration ...
    volumes:
      - '/dev/urandom:/dev/random:ro'  # Required for Metabase
      - 'duckdb-data:/duckdb-data'     # Volume for DuckDB files
      
volumes:
  postgresql-data:
    name: metabase-postgresql-data
  duckdb-data:
    name: metabase-duckdb-data
```

When connecting to DuckDB in Metabase, use the path `/duckdb-data/your-database-name.duckdb`.

### Bind Mounts for Local Development

For local development, you might prefer to use bind mounts to directly access files from your host machine:

```yaml
services:
  metabase:
    # ... other configuration ...
    volumes:
      - '/dev/urandom:/dev/random:ro'    # Required for Metabase
      - './duckdb:/duckdb-data'           # Bind mount local directory
```

Create a `duckdb` directory in the same location as your docker-compose.yaml file, and all DuckDB files will be stored there.

## Working with DuckDB Files

### Creating a New DuckDB Database

When configuring a new DuckDB connection in Metabase:

1. Use the path `/duckdb-data/new-database.duckdb`
2. Metabase will create the file if it doesn't exist

### Using an Existing DuckDB Database

To use an existing DuckDB database:

1. Place the .duckdb file in your mounted volume directory
2. Connect using the appropriate path (e.g., `/duckdb-data/existing-database.duckdb`)

### Importing Data

For importing data into DuckDB, you can create a separate volume or bind mount:

```yaml
services:
  metabase:
    # ... other configuration ...
    volumes:
      - '/dev/urandom:/dev/random:ro'
      - 'duckdb-data:/duckdb-data'
      - './import-data:/import-data:ro'  # Read-only mount for import data
```

This allows you to import data using DuckDB's import functions:

```sql
-- Import CSV data
CREATE TABLE users AS 
SELECT * FROM read_csv_auto('/import-data/users.csv');

-- Import Parquet data
CREATE TABLE products AS 
SELECT * FROM read_parquet('/import-data/products.parquet');
```

## Backup and Restore

### Backing Up DuckDB Files

DuckDB databases are standalone files, making them easy to back up:

```bash
# From the host machine, if using a bind mount
cp ./duckdb/my-database.duckdb ./duckdb/my-database-backup.duckdb

# Or using Docker commands with a named volume
docker run --rm -v metabase-duckdb-data:/data -v $(pwd):/backup alpine cp /data/my-database.duckdb /backup/my-database-backup.duckdb
```

### Backing Up PostgreSQL Data

To back up Metabase's PostgreSQL data:

```bash
docker exec -t metabase-duckdb_postgresql_1 pg_dumpall -c -U metabase_user > backup.sql
```

### Restoring from Backup

To restore a DuckDB database:

```bash
# With bind mount
cp ./my-database-backup.duckdb ./duckdb/my-database.duckdb

# With named volume
docker run --rm -v metabase-duckdb-data:/data -v $(pwd):/backup alpine cp /backup/my-database-backup.duckdb /data/my-database.duckdb
```

To restore PostgreSQL data:

```bash
cat backup.sql | docker exec -i metabase-duckdb_postgresql_1 psql -U metabase_user
```

## Volume Maintenance

### Inspecting Volumes

View information about your volumes:

```bash
docker volume ls
docker volume inspect metabase-duckdb-data
```

### Cleaning Up Unused Volumes

Remove unused volumes:

```bash
docker volume prune
```

Use caution with this command, as it will remove all volumes not used by at least one container.

### Volume Permissions

If you encounter permission issues, you may need to adjust permissions on your bind mounts:

```bash
# Make directory and contents accessible to Docker container
chmod -R 777 ./duckdb
```

For production environments, use a more restricted permission set according to your security requirements.

## Production Considerations

For production deployments:

1. **Use named volumes** instead of bind mounts for better portability
2. **Set up regular backups** of your data volumes
3. **Consider volume drivers** for cloud deployments (e.g., AWS EBS, Azure Disk)
4. **Implement monitoring** to track volume usage and performance
5. **Set appropriate permissions** for security

Example production-ready docker-compose.yaml volumes configuration:

```yaml
services:
  metabase:
    # ... other configuration ...
    volumes:
      - '/dev/urandom:/dev/random:ro'
      - 'duckdb-data:/duckdb-data'
    user: "1000:1000"  # Run as non-root user
      
volumes:
  postgresql-data:
    name: metabase-postgresql-data
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/path/to/persistent/storage/postgres'
  duckdb-data:
    name: metabase-duckdb-data
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/path/to/persistent/storage/duckdb'
```

## Troubleshooting Volume Issues

### Cannot Access DuckDB File

If Metabase cannot access the DuckDB file:

1. Check file permissions on the host directory
2. Verify the path used in the connection string
3. Confirm the volume is correctly mounted

### Data Not Persisting

If data isn't persisting between container restarts:

1. Ensure you're using named volumes or bind mounts correctly
2. Check that the volume paths in docker-compose.yaml are correct
3. Verify that the DuckDB connection in Metabase is using the correct path

### Volume Performance Issues

If you experience slow performance:

1. Use local volumes instead of network-based storage for DuckDB files
2. Consider using volume drivers optimized for your environment
3. Monitor I/O performance and adjust as needed
