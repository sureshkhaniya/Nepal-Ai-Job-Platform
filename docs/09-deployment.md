# Deployment Strategy

The exact cloud vendor is intentionally not locked in during planning. The MVP needs a simple, secure deployment model rather than premature Kubernetes complexity.

## Deployable units

1. Next.js web application
2. Node.js/TypeScript API
3. BullMQ AI/background worker
4. PostgreSQL
5. Redis
6. Private object storage

Web/API/worker should be independently deployable and horizontally scalable where the selected hosting platform supports it.

## Environments

### Local
Docker-backed PostgreSQL/Redis and safe development integrations.

### Staging
Production-like configuration using non-production OAuth/SMS/eSewa/AI credentials. Used for end-to-end and payment UAT.

### Production
Isolated credentials, production database/storage, backups, monitoring, and restricted operational access.

## Configuration

All environment-specific configuration is supplied outside source control. `.env.example` documents required variable names without values.

Expected groups:
- application URLs
- PostgreSQL
- Redis
- auth/session secrets
- Google OAuth
- SMS provider
- object storage
- AI provider
- eSewa
- observability

## CI/CD

Target pipeline:

```text
Pull Request
  -> lint/typecheck
  -> unit/integration tests
  -> production builds
  -> migration validation
  -> review

main
  -> build immutable artifacts
  -> deploy staging/production according to release policy
  -> run safe migrations
  -> health verification
```

Production database migrations should be forward-compatible with rolling deployments when possible.

## Health endpoints

- liveness: process is running
- readiness: process can serve its responsibilities

Do not expose sensitive dependency information publicly in health responses.

## Backups

Before launch:
- automated PostgreSQL backups enabled
- retention defined
- restore procedure documented and tested
- object storage durability understood

## Observability

Track:
- HTTP latency/error rate
- database connection saturation and slow queries
- Redis availability
- queue depth and oldest waiting job
- worker failure/dead-letter rate
- AI provider latency/cost/cache hit ratio
- SMS failures
- eSewa verification/reconciliation failures
- storage errors

## Scaling order

1. Measure bottleneck.
2. Optimize inefficient queries/indexes.
3. Scale stateless web/API instances.
4. Scale worker concurrency independently.
5. Tune PostgreSQL connection pooling.
6. Introduce additional infrastructure only when measured load requires it.

## Release readiness

Production launch requires HTTPS, domain/DNS, secure headers, secrets configured, migrations applied, backups tested, admin account/bootstrap procedure, eSewa production verification, SMS production verification, error monitoring, smoke tests, and rollback/recovery instructions.
