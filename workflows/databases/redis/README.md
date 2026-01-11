# Redis Validation Workflows

Workflows for validating Redis server installations and configurations.

## Available Workflows

| Workflow | Description |
|----------|-------------|
| `redis-connectivity` | Basic connectivity check with PING and GET/SET operations |

## Configuration Parameters

### redis-connectivity

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `host` | string | `redis.default.svc.cluster.local` | Redis host address |
| `port` | string | `6379` | Redis port |
| `password` | string | _(empty)_ | Redis password (sensitive) |
| `db` | string | `0` | Redis database number |

## Usage Examples

### Basic Usage

```bash
# Using default configuration (redis in default namespace)
testkube run testworkflow redis-connectivity
```

### Custom Configuration

```bash
# Connect to Redis in a different namespace
testkube run testworkflow redis-connectivity \
  --config host=redis.myapp.svc.cluster.local

# Connect with authentication
testkube run testworkflow redis-connectivity \
  --config host=redis.production.svc.cluster.local \
  --config password=mysecretpassword

# Connect to a specific database
testkube run testworkflow redis-connectivity \
  --config host=redis.default.svc.cluster.local \
  --config db=5
```

### Using with Secrets

For production environments, use Kubernetes secrets for the password:

```bash
# Create a secret
kubectl create secret generic redis-credentials \
  --from-literal=password=mysecretpassword

# Reference in workflow config (requires workflow modification)
```

## What Gets Validated

The `redis-connectivity` workflow performs:

1. **PING Test** - Verifies basic connectivity and authentication
2. **SET Operation** - Tests write capability
3. **GET Operation** - Tests read capability and data integrity
4. **Cleanup** - Removes test data
5. **Server Info** - Retrieves basic server information

## Troubleshooting

### Connection Refused

- Verify the Redis service is running: `kubectl get svc redis`
- Check the host and port configuration
- Ensure network policies allow traffic from Testkube namespace

### Authentication Failed

- Verify the password is correct
- Check if Redis requires authentication (`requirepass` setting)
- Ensure the password doesn't contain special characters that need escaping

### Timeout

- Check if Redis is overloaded
- Verify network connectivity between namespaces
- Consider increasing resource limits on Redis
