# DuckDB Integration Guide

This guide explains how to use DuckDB with Metabase in this Docker image.

## What is DuckDB?

[DuckDB](https://duckdb.org/) is an in-process analytical database management system, similar to SQLite but designed for OLAP (analytical) workloads rather than OLTP (transactional) workloads. DuckDB offers several advantages including:

- Fast analytical queries
- Easy integration with data science workflows
- Columnar storage format optimized for analytics
- SQL compatibility
- Support for complex analytical functions
- Minimal configuration requirements

## How This Docker Image Integrates DuckDB

This Docker image includes:

1. Metabase v0.52.9
2. The DuckDB driver v0.3.0 from MotherDuck

The DuckDB driver is pre-installed in the `/home/plugins/` directory, which means that DuckDB is available as a database option as soon as you start Metabase.

## Connecting to DuckDB Databases

### Basic Connection

1. Start Metabase and navigate to Admin Settings > Databases > Add Database
2. Select "DuckDB" from the dropdown list
3. Configure the connection:
   - Display name: Give your connection a name (e.g., "Analytics DuckDB")
   - Connection string: The path to your DuckDB database file

### Connection Options

There are several ways to connect to DuckDB:

#### In-Memory Database

```
:memory:
```

This creates a temporary in-memory database. Data will be lost when Metabase restarts.

#### File-Based Database

```
/path/to/your/database.duckdb
```

To use a file-based database, you need to mount the file or directory into the container:

```bash
docker run -d -p 3000:3000 -v /local/path/to/data:/data ghcr.io/yourusername/metabase-duckdb:latest
```

Then use `/data/database.duckdb` as the connection string.

#### Remote DuckDB (MotherDuck)

If you're using MotherDuck (the cloud version of DuckDB), you can connect using:

```
md:your_database_name
```

Note that you'll need to configure authentication with MotherDuck, typically by setting the `motherduck_token` parameter.

## Example Queries

Once connected, you can run SQL queries against your DuckDB database. Here are some examples:

### Creating Tables

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  name VARCHAR,
  email VARCHAR,
  created_at TIMESTAMP
);

INSERT INTO users VALUES 
  (1, 'Alice', 'alice@example.com', CURRENT_TIMESTAMP),
  (2, 'Bob', 'bob@example.com', CURRENT_TIMESTAMP);
```

### Analytical Queries

```sql
-- Simple aggregation
SELECT 
  DATE_TRUNC('month', created_at) AS month,
  COUNT(*) AS new_users
FROM users
GROUP BY month
ORDER BY month;

-- Window functions
SELECT 
  name,
  email,
  ROW_NUMBER() OVER (ORDER BY created_at) AS user_sequence
FROM users;
```

For a more comprehensive set of example queries and dashboard ideas, see the [Example Queries](./EXAMPLE_QUERIES.md) document.

## Performance Optimization

For optimal performance with DuckDB in Metabase:

1. **Create indexes** on frequently queried columns
2. **Use appropriate data types** for your columns
3. **Partition large datasets** when appropriate
4. **Limit result sets** to prevent memory issues
5. **Create materialized views** for complex queries that are run frequently

## Importing Data into DuckDB

DuckDB excels at importing data from various sources:

### From CSV

```sql
CREATE TABLE transactions AS 
SELECT * FROM read_csv_auto('/data/transactions.csv');
```

### From Parquet

```sql
CREATE TABLE products AS 
SELECT * FROM read_parquet('/data/products.parquet');
```

### From JSON

```sql
CREATE TABLE customers AS 
SELECT * FROM read_json_auto('/data/customers.json');
```

## Limitations and Considerations

When using DuckDB with Metabase, be aware of these limitations:

1. **Concurrency**: DuckDB has limited support for concurrent writes
2. **Memory Usage**: In-memory databases are limited by available RAM
3. **Driver Version**: The included driver version (0.3.0) might not support all features of the latest DuckDB release
4. **Query Complexity**: Very complex analytical queries might require optimization

## Troubleshooting DuckDB Connections

If you encounter issues connecting to DuckDB:

1. **Check the file path**: Ensure the DuckDB file is accessible within the container
2. **Verify permissions**: Make sure the container has read/write access to the DuckDB file
3. **Check logs**: Examine Metabase logs for any DuckDB-related errors
4. **Version compatibility**: Ensure your DuckDB file is compatible with the driver version

## Updating the DuckDB Driver

If you need to update the DuckDB driver to a newer version:

1. Modify the Dockerfile to use the updated driver URL
2. Rebuild the Docker image
3. Test thoroughly before deploying to production

## Additional Resources

- [DuckDB Documentation](https://duckdb.org/docs/)
- [MotherDuck Metabase Driver Repository](https://github.com/motherduckdb/metabase_duckdb_driver)
- [Metabase Documentation](https://www.metabase.com/docs/latest/)
- [SQL Features Supported by DuckDB](https://duckdb.org/docs/sql/introduction)
