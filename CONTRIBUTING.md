# Contributing Guidelines

Thank you for your interest in contributing to the Testkube Infrastructure Validation Workflows repository!

## Table of Contents

- [Development Setup](#development-setup)
- [Adding a New Workflow](#adding-a-new-workflow)
- [Image Security Requirements](#image-security-requirements)
- [Metadata Requirements](#metadata-requirements)
- [Testing Your Workflow](#testing-your-workflow)
- [Pull Request Process](#pull-request-process)

---

## Development Setup

### Pre-commit Hooks (Recommended)

This repository uses [pre-commit](https://pre-commit.com/) to automatically run `yamllint` on workflow files before each commit.

**Install pre-commit:**

```bash
# macOS (using Homebrew)
brew install pre-commit

# Linux (using pip)
pip install pre-commit
```

**Enable the hooks in your local clone:**

```bash
pre-commit install
```

Now `yamllint` will automatically run on any modified workflow files when you commit. To manually run on all files:

```bash
pre-commit run --all-files
```

---

## Adding a New Workflow

### 1. Choose the Appropriate Category

Place your workflow in the correct category folder:

| Category | Components |
|----------|------------|
| `databases/` | PostgreSQL, MySQL, MongoDB, Redis (as DB), etc. |
| `messaging/` | Kafka, RabbitMQ, NATS, Pulsar, etc. |
| `caching/` | Redis, Memcached, Hazelcast, etc. |
| `storage/` | MinIO, Elasticsearch, OpenSearch, etc. |
| `networking/` | NGINX Ingress, Traefik, Istio, Linkerd, etc. |
| `observability/` | Prometheus, Grafana, Jaeger, Loki, etc. |
| `security/` | Vault, cert-manager, Keycloak, etc. |
| `other/` | Whatever doesn't fit in any of these categories |

> **Need a new category?** Open an issue to request additions to the approved list.

### 2. Create the Workflow Structure

```
workflows/<category>/<component>/
├── <component>-<validation-type>.yaml
└── README.md (optional)
```

### 3. Follow Naming Conventions

- Workflow names: `<component>-<validation-type>` (lowercase, kebab-case)
- Examples: `redis-connectivity`, `kafka-topic-validation`, `postgresql-replication`

---

## Image Security Requirements

**All workflows must comply with these security policies. PRs that violate these rules will be automatically rejected by CI.**

### Approved Image Sources

Only images from these registries are permitted:

| Registry | Description |
|----------|-------------|
| `docker.io/library/` | Official Docker Hub images |
| `docker.io/bitnami/` | Bitnami verified images |
| `cgr.dev/chainguard/` | Chainguard hardened images |
| `gcr.io/distroless/` | Google distroless images |
| `quay.io/prometheus/` | Official Prometheus images |
| `docker.io/grafana/` | Official Grafana images |
| `docker.io/curlimages/` | Official curl images |
| `docker.io/confluentinc/` | Official Confluent/Kafka images |

> **Need a new registry?** Open an issue to request additions to the approved list.

### Digest Pinning Required

For security reasons, all images **must** use SHA256 digest pinning instead of mutable tags:

```yaml
# ❌ NOT ALLOWED - tags can be overwritten
image: redis:7-alpine
image: postgres:16
image: curlimages/curl:latest

# ✅ REQUIRED - immutable digest reference
image: redis:7-alpine@sha256:abc123...
image: docker.io/library/postgres:16-alpine@sha256:def456...
```

### How to Get Image Digests

```bash
# Pull the image and get its digest
docker pull redis:7-alpine
docker inspect --format='{{index .RepoDigests 0}}' redis:7-alpine

# Or use crane (recommended)
crane digest redis:7-alpine

# Or check Docker Hub directly
# https://hub.docker.com/_/redis/tags
```

### Why These Requirements?

| Requirement | Threat Mitigated |
|-------------|------------------|
| Approved registries | Prevents use of unvetted/malicious images |
| Digest pinning | Prevents tag hijacking and ensures reproducibility |
| No `latest` tags | Ensures version consistency and auditability |

---

## Metadata Requirements

### Required Labels

Every workflow **must** include these labels:

```yaml
metadata:
  labels:
    catalog.testkube.io/category: "<category>"           # databases, messaging, etc.
    catalog.testkube.io/component: "<component>"         # redis, kafka, etc.
```

### Required Annotations

Every workflow **must** include these annotations:

```yaml
metadata:
  annotations:
    catalog.testkube.io/display-name: "Human Readable Name"
    catalog.testkube.io/description: "Clear description of what this workflow validates"
```

### Recommended Annotations

```yaml
metadata:
  annotations:
    catalog.testkube.io/icon: "https://cdn.simpleicons.org/<component>"  # Icon URL (see below)
```

#### Finding Icon URLs

Use [Simple Icons](https://simpleicons.org/) CDN for component icons:

1. Search for your component at [simpleicons.org](https://simpleicons.org/)
2. Use the URL pattern: `https://cdn.simpleicons.org/<slug>`

Examples:
- PostgreSQL: `https://cdn.simpleicons.org/postgresql`
- Redis: `https://cdn.simpleicons.org/redis`
- Apache Kafka: `https://cdn.simpleicons.org/apachekafka`
- MongoDB: `https://cdn.simpleicons.org/mongodb`
- RabbitMQ: `https://cdn.simpleicons.org/rabbitmq`

---

## Workflow Best Practices

### Configuration Parameters

- Make all connection details configurable via `spec.config`
- Provide sensible defaults (e.g., standard Kubernetes service names)
- Mark sensitive parameters with `sensitive: true`

```yaml
spec:
  config:
    host:
      type: string
      default: "redis.default.svc.cluster.local"
      description: "Redis host address"
    password:
      type: string
      default: ""
      sensitive: true
      description: "Redis password (optional)"
```

### Output and Logging

- Print clear status messages during validation
- Use checkmarks (✓) and crosses (✗) for pass/fail indication
- Include relevant details in error messages

### Cleanup

- Remove any test data created during validation
- Use temporary keys/resources with TTL when possible

---

## Testing Your Workflow

Before submitting a PR, verify your workflow:

### 1. YAML Validation

First, install `yamllint` if you don't have it:

```bash
# macOS (using Homebrew)
brew install yamllint

# Linux (Debian/Ubuntu)
sudo apt-get install yamllint

# Using pip (any platform)
pip install yamllint
```

Then validate your workflow (the `.yamllint.yaml` config file in the repo root will be used automatically):

```bash
# Validate YAML syntax
yamllint workflows/path/to/your-workflow.yaml
```

### 2. Image Compliance

```bash
# Check images are from approved registries and have digests
grep -h "image:" workflows/path/to/your-workflow.yaml
```

### 3. Local Testing

```bash
# Deploy to a test Testkube instance
testkube create testworkflow -f workflows/path/to/your-workflow.yaml

# Run the workflow
testkube run testworkflow your-workflow-name --watch
```

### PR Checklist

- [ ] Workflow is in the correct category folder
- [ ] Follows naming convention: `<component>-<validation-type>.yaml`
- [ ] All images are from approved registries
- [ ] All images use digest pinning (`@sha256:...`)
- [ ] Required labels are present
- [ ] Required annotations are present
- [ ] Config parameters have descriptions and defaults
- [ ] Workflow tested locally
- [ ] README.md updated (if adding new component)

---

## Pull Request Process

1. **Fork** the repository
2. **Create a branch** for your changes
3. **Add your workflow** following the guidelines above
4. **Test** your workflow locally
5. **Submit a PR** with a clear description
6. **CI checks** will validate image compliance automatically
7. **Review** - maintainers will review and provide feedback
8. **Merge** - once approved, your workflow will be merged

---

## Questions?

- Open an [Issue](../../issues) for questions or feature requests
- Check existing [Discussions](../../discussions) for common topics

Thank you for contributing! 🎉
