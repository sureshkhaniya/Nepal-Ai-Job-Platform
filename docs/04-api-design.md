# API and Background Worker Design

## API conventions

Base path: `/v1`

- JSON request/response for normal APIs.
- File upload endpoints use multipart only where required.
- Validate every request at the boundary.
- Authentication establishes identity; authorization separately checks role, ownership, company membership, resource state, and action.
- Use stable machine-readable error codes plus safe human messages.
- Generate a request/correlation ID for tracing.
- Pagination must be bounded.

## Representative endpoints

### Authentication

```text
POST /v1/auth/otp/request
POST /v1/auth/otp/verify
GET  /v1/auth/google/start
GET  /v1/auth/google/callback
POST /v1/auth/logout
GET  /v1/me
```

OTP request endpoints require phone/IP throttling and anti-enumeration responses.

### Candidate

```text
GET   /v1/candidate/profile
PATCH /v1/candidate/profile
POST  /v1/candidate/resumes
GET   /v1/candidate/resumes
DELETE /v1/candidate/resumes/:resumeId
GET   /v1/candidate/applications
```

### Jobs

```text
GET  /v1/jobs
GET  /v1/jobs/:jobIdOrSlug
POST /v1/jobs/:jobId/applications
```

### Employer

```text
POST  /v1/companies
GET   /v1/companies/:companyId
PATCH /v1/companies/:companyId
POST  /v1/companies/:companyId/jobs
PATCH /v1/jobs/:jobId
POST  /v1/jobs/:jobId/publish
POST  /v1/jobs/:jobId/close
GET   /v1/jobs/:jobId/applications
PATCH /v1/applications/:applicationId/status
```

### eSewa

```text
POST /v1/payments/esewa/initiate
GET  /v1/payments/esewa/success
GET  /v1/payments/esewa/failure
GET  /v1/payments/:paymentId
```

## Asynchronous application submit

`POST /v1/jobs/:jobId/applications` must not call an LLM synchronously.

Within one PostgreSQL transaction:

1. Authenticate candidate.
2. Validate that job is published and not expired/closed.
3. Validate that the candidate owns the selected resume.
4. Resolve the cached `parsed_resumes` record.
5. Insert `job_applications`.
6. Insert durable `ai_jobs` work record.
7. Insert transactional outbox event.
8. Commit.

Return immediately:

```http
HTTP/1.1 202 Accepted
```

```json
{
  "application_id": "uuid",
  "status": "AI_PROCESSING"
}
```

An outbox dispatcher publishes the event to BullMQ. This avoids a database/Redis dual-write race.

## Queue payload

Keep messages small and non-sensitive:

```json
{
  "schema_version": 1,
  "ai_job_id": "uuid",
  "application_id": "uuid"
}
```

Do not place resume plaintext, full job descriptions, access tokens, or payment secrets in Redis.

## Worker lifecycle

1. Consume job.
2. Load durable AI job/application from PostgreSQL.
3. Acquire idempotency/processing guard.
4. Load parsed resume text and canonical job description.
5. Check AI result cache using resume hash + job content hash + prompt version + model family.
6. If cache hit, validate/reuse result.
7. If miss, invoke provider with timeout.
8. Strictly validate structured response.
9. Persist evaluation and AI usage telemetry in a transaction.
10. Mark job complete.

## AI failure policy

Retry only transient conditions such as timeouts, connection resets, 408, 429, 502, 503, and 504 using exponential backoff with jitter. Do not blindly retry invalid credentials, invalid schemas, unsupported models, or permanent validation errors.

After retry exhaustion:
- mark AI work dead-lettered/failed
- mark application AI processing state failed
- keep the candidate application valid for human review
- expose operational recovery to administrators

## Error envelope

Example:

```json
{
  "error": {
    "code": "JOB_NOT_ACCEPTING_APPLICATIONS",
    "message": "This job is not accepting applications.",
    "request_id": "..."
  }
}
```

Never expose stack traces, SQL errors, provider secrets, or raw payment verification internals to clients.
