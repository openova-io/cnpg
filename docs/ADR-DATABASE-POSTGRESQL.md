# ADR: PostgreSQL with CloudNativePG

**Status:** Accepted
**Date:** 2024-04-01

## Context

Need managed PostgreSQL for relational data. Options: managed DBaaS, self-hosted operator, or raw StatefulSet.

## Decision

Use CloudNativePG (CNPG) operator for PostgreSQL management.

## Rationale

| Option | Pros | Cons |
|--------|------|------|
| Cloud DBaaS | Zero ops | Cost, vendor lock-in |
| **CNPG operator** | K8s-native, HA, backups | Self-managed | **Selected** |
| Raw StatefulSet | Simple | Manual HA, backups |

**Key Decision Factors:**
- Kubernetes-native lifecycle
- Automatic failover
- Built-in backup to S3-compatible storage
- No cloud vendor lock-in

## Features

| Feature | Support |
|---------|---------|
| Automatic failover | ✅ Primary election |
| Rolling updates | ✅ Zero-downtime |
| Backups | ✅ Barman to MinIO/R2 |
| Point-in-time recovery | ✅ WAL archiving |
| Connection pooling | ✅ PgBouncer |
| Monitoring | ✅ Prometheus metrics |

## Configuration

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: <tenant>-postgres
  namespace: databases
spec:
  instances: 3
  storage:
    size: 10Gi
  backup:
    barmanObjectStore:
      destinationPath: s3://backups/<tenant>/postgres
      endpointURL: http://minio.storage:9000
```

## Consequences

**Positive:** K8s-native, automatic HA, integrated backups, Prometheus metrics
**Negative:** Operator learning curve, self-managed infrastructure

## Related

- [SPEC-POSTGRESQL-CONFIGURATION](./SPEC-POSTGRESQL-CONFIGURATION.md)
- [BLUEPRINT-CNPG-CLUSTER](./BLUEPRINT-CNPG-CLUSTER.md)
