# System Architecture

## Architectural goals

- Clear separation of interactive web/API traffic and expensive background work
- Strong authorization boundaries
- Provider-independent AI integration
- Reliable payment verification
- Simple deployment suitable for a small team
- Ability to scale web/API and worker processes independently

## Logical components

```text
Browser
   |
   v
Web Application
   |
   v
API / Application Service -------------------+
   |             |                            |
   v             v                            v
PostgreSQL     Redis / Queue              Object/File Storage
                 |
                 v
             AI Worker
                 |
                 +--> Resume extraction
                 +--> Structured normalization
                 +--> AI provider
                 +--> Match persistence

External integrations:
- Google authentication
- OTP/SMS provider
- eSewa
- AI provider
- transactional email/notification provider (if selected)
```

## Application boundaries

### `apps/web`
Owns browser UI for candidates, employers, and administrators. It must not contain trusted authorization/payment logic that can be bypassed by direct API calls.

### `apps/api`
Owns core business rules, authentication/session integration, authorization, validation, persistence coordination, job/application operations, payment initiation/verification, and queue dispatch.

### `apps/ai-worker`
Consumes asynchronous tasks. It performs file extraction/normalization and AI matching, persists results through controlled data access, and exposes no public browser-facing endpoint unless operationally required.

## Shared packages

### `packages/database`
Schema/client, migrations helpers, repositories/query utilities, and transaction boundaries.

### `packages/auth`
Shared authentication types/policies and authorization helpers. Provider-specific adapters should remain isolated.

### `packages/validation`
Shared schemas for API boundaries and domain primitives.

### `packages/localization`
Translation resources and Nepal-specific locale helpers.

### `packages/shared`
Small dependency-light shared types/utilities. Avoid turning this into an unstructured dumping ground.

## Data stores

### PostgreSQL
System of record for users, organizations, jobs, applications, resume metadata, match results, payments, entitlements, audit data, and relevant reference data.

Use relational columns for frequently constrained/searched values. Use JSONB for genuinely flexible structured payloads such as versioned extraction/match details—not as a replacement for database design.

### Redis
Used for ephemeral caching, rate-limit primitives where selected, and queue coordination. Redis is not the authoritative store for business records.

### File/object storage
Original resume files should live in private storage rather than public web paths. Access should use authorization-checked application flows or short-lived signed access where supported.

## Asynchronous processing

API request:
1. Validate upload metadata/content.
2. Store file securely.
3. Calculate SHA-256.
4. Persist resume record.
5. Check reusable extraction cache according to version/policy.
6. Enqueue work when processing is required.
7. Return processing state immediately.

Worker:
1. Claim job.
2. Mark processing state safely.
3. Extract text locally using format-specific tooling.
4. Normalize/sanitize extracted content.
5. Persist versioned extraction.
6. Generate match work when required.
7. Call configured AI provider with minimized necessary data.
8. Validate provider output against a strict schema.
9. Persist match result/version.
10. Mark completion or recoverable failure.

Queue operations should be retry-safe. Job identifiers should allow duplicate delivery without duplicate business effects.

## AI boundary

AI is advisory. The application should define an internal interface such as a match service rather than spreading provider calls throughout controllers/UI.

Inputs should be explicit and minimized. Outputs should be schema validated and include model/prompt or algorithm version metadata needed for reproducibility and debugging.

A provider failure results in an unavailable/pending match—not a failed job application.

## Security boundaries

Every protected API operation evaluates:
1. authenticated actor
2. actor role
3. resource ownership/organization membership
4. requested action
5. resource state

The UI hiding a button is never considered authorization.

## Payment boundary

Pricing/plan identity is resolved on the server. eSewa responses are verified according to the integration contract. A verified transaction is converted into an internal payment state and entitlement through an idempotent transaction.

## Deployment shape

Initial production can run as separately deployable processes/services:
- web
- API
- worker
- PostgreSQL
- Redis
- private file/object storage

The exact hosting provider will be chosen later. The architecture must not require Kubernetes for MVP.

## Observability

At minimum capture:
- structured application errors
- request correlation identifiers
- worker job identifiers/status
- queue depth/failure signals
- payment verification failures
- authentication abuse signals
- AI provider latency/failure without logging sensitive prompt contents indiscriminately

## Architecture decision rule

Prefer the simplest design that preserves security, data integrity, asynchronous processing, and future replaceability. New infrastructure requires a concrete problem, not hypothetical scale.
