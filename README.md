# Testkube Infrastructure Validation Workflows

A community-driven collection of reusable [Testkube](https://testkube.io) workflows for validating infrastructure components in Kubernetes environments.

## Overview

This repository contains pre-built TestWorkflows that validate the configuration and health of common infrastructure components like databases, message brokers, caching systems, and more.

**Key Features:**
- 🔌 **Plug-and-play** - Configure and run in minutes
- 🔒 **Security-first** - All images vetted and digest-pinned
- 🏷️ **Well-categorized** - Easy to browse and discover
- 🤝 **Community-driven** - Open for contributions

## Repository Structure

```
workflows/
├── databases/          # PostgreSQL, MySQL, MongoDB, Redis
├── messaging/          # Kafka, RabbitMQ, NATS
├── caching/            # Redis, Memcached
├── storage/            # MinIO, Elasticsearch
├── networking/         # Ingress controllers, Service mesh
├── observability/      # Prometheus, Grafana, Jaeger
└── security/           # Vault, cert-manager
```

## Quick Start

### 1. Browse Available Workflows

Explore the `workflows/` directory to find validation workflows for your infrastructure components.

### 2. Deploy a Workflow

```bash
# Apply directly to your Testkube instance
kubectl apply -f workflows/databases/redis/redis-connectivity.yaml

# Or use the Testkube CLI
testkube create testworkflow -f workflows/databases/redis/redis-connectivity.yaml
```

### 3. Run the Workflow

```bash
# Run with default configuration
testkube run testworkflow redis-connectivity

# Run with custom parameters
testkube run testworkflow redis-connectivity \
  --config host=my-redis.default.svc.cluster.local \
  --config port=6379
```

## Workflow Metadata

All workflows include standardized metadata for easy discovery:

### Labels (for filtering)

| Label | Purpose | Values |
|-------|---------|--------|
| `testkube.io/category` | Infrastructure type | `databases`, `messaging`, `caching`, `storage`, `networking`, `observability`, `security` |
| `testkube.io/component` | Specific component | `redis`, `postgresql`, `kafka`, etc. |
| `testkube.io/validation-type` | What's being validated | `connectivity`, `health`, `performance`, `security` |

### Annotations (for display)

| Annotation | Purpose |
|------------|---------|
| `testkube.io/display-name` | Human-readable name |
| `testkube.io/description` | What the workflow validates |
| `testkube.io/icon` | Icon identifier for UIs |
| `testkube.io/tags` | Search keywords |

## Security

All workflows in this repository follow strict security guidelines:

- ✅ **Approved registries only** - Images from vetted sources
- ✅ **Digest pinning** - Immutable image references
- ✅ **Automated validation** - CI checks on every PR

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Contributing

We welcome contributions! Please read our [Contributing Guidelines](CONTRIBUTING.md) before submitting a PR.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.
