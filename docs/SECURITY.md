# Security Best Practices

This document outlines security best practices for deploying and using the Metabase-DuckDB Docker image.

## Secure Configuration

### Authentication and Authorization

- **Change Default Credentials**: Always change the default credentials for both Metabase and PostgreSQL
- **Use Strong Passwords**: Use complex passwords for all accounts
- **Use Environment Variables**: Store sensitive information in environment variables or Docker secrets rather than hardcoding them
- **Enable Metabase SSO**: Consider setting up Single Sign-On with your identity provider

### Network Security

- **Use Internal Networks**: When using Docker Compose, avoid exposing the PostgreSQL port to the host unless necessary
- **Set Up HTTPS**: Configure a reverse proxy (like Nginx) with SSL/TLS to secure the connection to Metabase
- **Use a Firewall**: Restrict access to your Metabase instance with a firewall

### Container Security

- **Keep Images Updated**: Regularly update to the latest version of this image to receive security patches
- **Run as Non-Root**: The container is already configured to run as a non-root user, don't override this
- **Limit Resource Usage**: Set memory and CPU limits to prevent resource exhaustion attacks
- **Use Read-Only Filesystem**: When possible, mount filesystems as read-only
- **Scan for Vulnerabilities**: Use tools like Trivy or Clair to scan the image for vulnerabilities

## Data Security

### Database Connections

- **Use Least Privilege Accounts**: Create database users with minimal necessary permissions
- **Encrypt Connections**: Use SSL/TLS for database connections whenever possible
- **Rotate Credentials**: Regularly rotate database credentials

### Query Security

- **Sanitize Inputs**: Be careful with user-generated SQL queries
- **Implement Row-Level Security**: Use database features to restrict data access at the row level
- **Audit Access**: Set up logging and auditing to track who accesses what data

## Operational Security

### Logging and Monitoring

- **Enable Metabase Audit Logs**: Configure audit logging to track user actions
- **Monitor Container Health**: Set up monitoring for the container's health and resource usage
- **Log Retention**: Establish a log retention policy that complies with your requirements

### Backups

- **Regular Backups**: Set up regular backups of your PostgreSQL database
- **Test Restoration**: Periodically test your backup restoration process
- **Secure Backup Storage**: Store backups securely and consider encryption

### Updates and Patches

- **Automated Updates**: Consider setting up automated image updates with vulnerability scanning
- **Change Management**: Establish a process for testing and deploying updates
- **Version Control**: Keep track of which image versions you're using

## Docker Specific Security

### Registry Security

- **Use Official Images**: Always pull base images from official repositories
- **Sign Images**: Consider signing your images for verification
- **Private Registry**: Use a private registry with access controls for sensitive environments

### Compose File Security

- **Don't Hardcode Secrets**: Use `.env` files or Docker secrets instead of hardcoding credentials
- **Limit Capabilities**: Avoid privileged mode and only add necessary capabilities
- **Network Segmentation**: Use dedicated networks for your application components

## Example: Secure Docker Compose Configuration

```yaml
services:
  metabase:
    image: ghcr.io/yourusername/metabase-duckdb:latest
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - '127.0.0.1:3000:3000'  # Only expose locally, use a reverse proxy
    volumes:
      - '/dev/urandom:/dev/random:ro'
    environment:
      - MB_DB_TYPE=postgres
      - MB_DB_HOST=postgresql
      - MB_DB_PORT=5432
      - MB_DB_DBNAME=metabase
      - MB_DB_USER_FILE=/run/secrets/db_user
      - MB_DB_PASS_FILE=/run/secrets/db_password
      - JAVA_TOOL_OPTIONS=-Xmx1g
    mem_limit: 1g
    mem_reservation: 512m
    read_only: true  # Read-only filesystem
    tmpfs:
      - /tmp:rw,noexec,nosuid
    healthcheck:
      test: 'curl --fail -I http://127.0.0.1:3000/api/health || exit 1'
      interval: 5s
      timeout: 20s
      retries: 10
    depends_on:
      postgresql:
        condition: service_healthy
    secrets:
      - db_user
      - db_password
    security_opt:
      - no-new-privileges:true
    networks:
      - metabase_frontend
      - metabase_backend

  postgresql:
    image: 'postgres:16-alpine'
    platform: linux/amd64
    restart: unless-stopped
    expose:
      - '5432'  # Only expose to internal network
    volumes:
      - 'postgresql-data:/var/lib/postgresql/data'
    environment:
      - POSTGRES_USER_FILE=/run/secrets/db_user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
      - POSTGRES_DB=metabase
    healthcheck:
      test:
        - CMD-SHELL
        - 'pg_isready -U $$(cat /run/secrets/db_user) -d metabase'
      interval: 5s
      timeout: 20s
      retries: 10
    secrets:
      - db_user
      - db_password
    security_opt:
      - no-new-privileges:true
    networks:
      - metabase_backend

volumes:
  postgresql-data:
    name: metabase-postgresql-data

secrets:
  db_user:
    file: ./secrets/db_user.txt  # File containing username
  db_password:
    file: ./secrets/db_password.txt  # File containing password

networks:
  metabase_frontend:
    name: metabase_frontend
  metabase_backend:
    name: metabase_backend
    internal: true  # Not accessible from outside
```

## Compliance Considerations

- **GDPR**: Ensure your Metabase setup complies with GDPR if processing EU citizens' data
- **HIPAA**: If handling healthcare data, ensure compliance with HIPAA requirements
- **PCI DSS**: Follow PCI DSS guidelines if processing payment card information
- **Data Residency**: Be mindful of where your data is stored and processed

## Security Incident Response

1. **Preparation**: Have a plan for responding to security incidents
2. **Containment**: Know how to quickly isolate compromised containers
3. **Eradication**: Remove compromised components and fix vulnerabilities
4. **Recovery**: Restore from known good backups
5. **Lessons Learned**: Document incidents and improve security measures
