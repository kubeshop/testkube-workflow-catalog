# Kafka Validation Workflows

Workflows for validating Apache Kafka broker installations and configurations.

## Available Workflows

| Workflow | Description |
|----------|-------------|
| `kafka-connectivity` | Basic connectivity check with topic listing and produce/consume |

## Configuration Parameters

### kafka-connectivity

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `bootstrap_servers` | string | `kafka.default.svc.cluster.local:9092` | Kafka bootstrap servers |
| `security_protocol` | string | `PLAINTEXT` | Security protocol |
| `test_topic` | string | `testkube-validation` | Topic for produce/consume test |

## Usage Examples

### Basic Usage

```bash
# Using default configuration
testkube run testworkflow kafka-connectivity
```

### Custom Configuration

```bash
# Connect to Kafka in a different namespace
testkube run testworkflow kafka-connectivity \
  --config bootstrap_servers=kafka.myapp.svc.cluster.local:9092

# Connect to multiple brokers
testkube run testworkflow kafka-connectivity \
  --config bootstrap_servers=kafka-0:9092,kafka-1:9092,kafka-2:9092

# Use a specific test topic
testkube run testworkflow kafka-connectivity \
  --config test_topic=my-test-topic
```

## What Gets Validated

The `kafka-connectivity` workflow performs:

1. **Broker Connectivity** - Verifies connection to bootstrap servers
2. **List Topics** - Retrieves available topics from the cluster
3. **Cluster Info** - Gets broker API versions and cluster metadata
4. **Test Topic** - Creates or verifies test topic exists
5. **Produce Message** - Sends a test message to the topic
6. **Consume Message** - Reads back the message to verify end-to-end

## Supported Configurations

### Security Protocols

| Protocol | Description |
|----------|-------------|
| `PLAINTEXT` | No encryption or authentication |
| `SSL` | TLS encryption |
| `SASL_PLAINTEXT` | SASL authentication without encryption |
| `SASL_SSL` | SASL authentication with TLS encryption |

> **Note:** For SASL/SSL configurations, additional workflow modifications may be needed to provide certificates and credentials.

## Troubleshooting

### Connection Refused

- Verify Kafka brokers are running: `kubectl get pods -l app=kafka`
- Check the bootstrap server addresses and ports
- Ensure network policies allow traffic from Testkube namespace

### Topic Creation Failed

- Check if auto-topic creation is enabled on the cluster
- Verify the user has permissions to create topics
- The workflow will still pass basic connectivity tests

### Consumer Timeout

- This may be normal if the topic is empty
- Check if there are active consumers in other consumer groups
- Verify topic has data with: `kafka-console-consumer.sh --from-beginning`

### SSL/SASL Errors

- Verify certificates are properly configured
- Check SASL mechanism matches broker configuration
- Ensure credentials are correct
