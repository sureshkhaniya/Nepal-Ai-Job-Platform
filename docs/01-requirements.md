# MVP Product Requirements

## 1. Candidate experience

### Account and identity
- Register/sign in using supported authentication methods.
- Support Google authentication.
- Support Nepal phone numbers (`+977`) with OTP verification where configured.
- Maintain secure sessions and account recovery/verification flows.

### Candidate profile
- Name and contact details
- Professional headline/summary
- Skills
- Work experience
- Education
- Preferred job categories/types
- Expected salary where applicable
- Nepal location fields
- Language preferences

Candidates control profile updates and resume visibility subject to application requirements.

### Resume management
- Accept supported PDF/DOCX uploads with strict size/type limits.
- Compute SHA-256 fingerprint of resume bytes.
- Reuse safely cacheable extraction results for identical content.
- Store original file metadata separately from extracted structured information.
- Display processing state: queued, processing, completed, failed.

### Job discovery
Candidates can browse/search/filter active jobs using relevant fields such as keyword, location, category, employment type, experience, remote/on-site preference, and salary where available.

### Applications
- Candidate applies to an active job.
- Prevent unintended duplicate applications according to business policy.
- Record application timestamp and status history.
- Employer-side status changes are visible to the candidate where appropriate.

### AI match insight
Where available, display an advisory match result including:
- normalized score
- strengths / matching evidence
- gaps or missing requirements
- concise explanation
- processing state

The score is not a hiring decision.

## 2. Employer experience

### Organization
- Employer users can create/join an organization according to platform policy.
- Company fields include identity, description, industry, website, size, and Nepal location.
- Employer permissions must be enforced server-side.

### Jobs
Employer can create/edit/publish/close vacancies with:
- title
- description
- responsibilities
- required/preferred skills
- experience level
- education requirements when applicable
- location / remote mode
- employment type
- salary information where provided
- application deadline
- status

### Applicant tracking
Employer can:
- list applicants for its jobs
- inspect candidate/application information permitted by privacy rules
- see resume processing/match status
- move an application through defined stages
- record internal notes where implemented
- filter/sort applicants using deterministic attributes and advisory match scores

## 3. Administration

Admins require protected tooling to manage:
- users
- employer organizations
- jobs
- reported/moderated content
- payment records/entitlements
- Nepal reference data where appropriate
- operational processing failures

Sensitive admin actions should be auditable.

## 4. Localization

- Critical UX supports English and Nepali.
- Translation keys, not duplicated page logic, drive localization.
- User-generated content is not automatically translated unless explicitly designed later.
- Nepal geography supports province, district, municipality/local level, and ward where the source dataset permits.

## 5. Payments

Initial paid employer capabilities will use eSewa where applicable.

Requirements:
- initiate payment from server-trusted pricing data
- never trust client-provided successful-payment claims
- verify callbacks/responses server-side
- make payment processing idempotent
- record transaction state transitions
- grant entitlement only after verified success
- preserve enough metadata for support/reconciliation

## 6. Notifications

MVP may support transactional notifications for high-value events such as verification, application status changes, employer activity, and payment outcomes. Notification delivery failures must not corrupt core business transactions.

## 7. Security requirements

- Role/ownership authorization on every protected backend operation
- Strict request validation
- Upload type/size validation
- Rate limiting for abuse-prone endpoints
- OTP brute-force protections
- Secure cookies/tokens
- Secrets exclusively through environment/secret management
- No credentials committed to Git
- Safe logging without passwords, OTPs, tokens, full sensitive resume content, or payment secrets
- Audit trail for important privileged actions

## 8. Reliability requirements

- AI/resume processing occurs asynchronously.
- Queue tasks are retryable and idempotent where practical.
- A failed AI provider must not prevent normal job browsing or application creation.
- Database operations use transactions for invariants that span multiple writes.
- Payment verification is idempotent.

## 9. Accessibility and responsive design

Critical candidate/employer flows should be keyboard-accessible, have semantic labels, reasonable contrast, useful validation messages, and work on common mobile and desktop viewport sizes.

## 10. Acceptance boundary

The MVP is ready for controlled launch when the end-to-end candidate and employer journeys work, critical security boundaries are tested, payment verification is tested, background processing is observable/recoverable, admin moderation exists, and production deployment/backup procedures are documented.
