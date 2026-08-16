# Testing Strategy

## Testing pyramid

### Unit tests
Fast tests for domain rules, validators, authorization policies, Nepal phone normalization, payment signature helpers, cache-key generation, and AI schema validation.

### Integration tests
Use real PostgreSQL/Redis test services where useful for:
- constraints and transactions
- application creation
- transactional outbox
- queue idempotency
- company authorization
- payment state transitions
- resume cache behavior

### End-to-end tests
Cover a small set of critical journeys:
1. candidate registration -> profile -> resume -> application
2. employer registration -> company -> job -> applicant review
3. AI job queued -> completed result visible
4. eSewa success verification -> entitlement
5. payment replay -> no duplicate entitlement
6. cross-company unauthorized access -> denied

## AI testing

Do not rely exclusively on live provider calls in CI.

- Mock provider for deterministic contract tests.
- Validate malformed output handling.
- Test timeout/429/5xx retry classification.
- Test retry exhaustion/dead-letter state.
- Maintain a small de-identified evaluation fixture set for match quality regression.
- Test that score stays within 0-100 and arrays are bounded.
- Verify candidate application remains valid when AI fails.

## Resume testing

Fixtures should include:
- normal PDF
- normal DOCX
- duplicate binary upload
- empty/corrupt file
- unsupported type
- oversized file
- extraction failure
- scanned PDF fallback case if OCR is implemented

Assert that identical SHA-256 input does not repeat local extraction unnecessarily.

## Payment testing

Use eSewa UAT/sandbox capabilities available at implementation time.

Test:
- valid success
- failure/cancellation
- invalid signature/response
- wrong amount
- wrong transaction UUID
- duplicate callback
- provider timeout
- pending transaction reconciliation
- already-paid replay

Never run destructive production payment tests in automated CI.

## Security testing

Critical authorization matrices must be automated. Test candidate A vs candidate B resources, company A employer vs company B resources, ordinary user vs admin endpoints, and inactive/revoked memberships.

## Performance testing

Before launch measure:
- public job listing/search latency
- job detail SSR/API latency
- application submit latency excluding async AI
- employer ATS list/sort latency
- queue throughput and oldest-job delay
- database query plans for high-volume endpoints

## CI gates

Pull requests should eventually require:
- formatting/lint
- TypeScript checks
- unit tests
- integration tests for affected backend areas
- production build
- migration validation
- dependency/security checks as configured

## Definition of done

Every feature PR includes tests proportional to risk. Payment, authentication, authorization, database invariants, and background job idempotency require automated coverage before being considered complete.
