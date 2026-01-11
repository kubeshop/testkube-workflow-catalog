# PostgreSQL Validation Workflows

Workflows for validating PostgreSQL server installations and configurations.

## Available Workflows

| Workflow | Description |
|----------|-------------|
| `postgresql-connectivity` | Basic connectivity check with query execution |

## Configuration Parameters

### postgresql-connectivity

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `host` | string | `postgresql.default.svc.cluster.local` | PostgreSQL host address |
| `port` | string | `5432` | PostgreSQL port |
| `database` | string | `postgres` | Database name to connect to |
| `username` | string | `postgres` | PostgreSQL username |
| `password` | string | _(empty)_ | PostgreSQL password (sensitive) |

## Usage Examples

### Basic Usage

```bash
# Using default configuration
testkube run testworkflow postgresql-connectivity
```

### Custom Configuration

```bash
# Connect to PostgreSQL in a different namespace
testkube run testworkflow postgresql-connectivity \
  --config host=postgresql.myapp.svc.cluster.local

# Connect with full credentials
testkube run testworkflow postgresql-connectivity \
  --config host=postgresql.production.svc.cluster.local \
  --config database=myapp \
  --config username=myuser \
  --config password=mypassword
```

## What Gets Validated

The `postgresql-connectivity` workflow performs:

1. **Connection Test** - Verifies basic TCP connectivity and authentication
2. **Server Version** - Retrieves PostgreSQL version information
3. **Write Operation** - Creates a temp table and inserts data
4. **Read Operation** - Reads back data to verify integrity
5. **Database Stats** - Shows current database size and connections

## Troubleshooting

### Connection Refused

- Verify the PostgreSQL service is running: `kubectl get svc postgresql`
- Check the host and port configuration
- Ensure `pg_hba.conf` allows connections from Testkube namespace

### Authentication Failed

- Verify username and password are correct
- Check PostgreSQL authentication method (md5, scram-sha-256, trust)
- Ensure the user has permission to access the specified database

### Permission Denied

- Verify the user has CREATE privileges for temp tables
- Check if the database exists and user has CONNECT privilege
