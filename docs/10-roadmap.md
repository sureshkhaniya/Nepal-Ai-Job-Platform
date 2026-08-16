# 12-Week MVP Roadmap

Target window: approximately 3 months. Dates can be attached once the formal project start date is confirmed.

## Phase 1 — Authentication & Schema (Weeks 1-3)

### Week 1: Foundation
- pnpm monorepo
- Next.js web shell
- Node.js/TypeScript API
- PostgreSQL + Redis local Docker
- shared configuration/validation
- migrations
- CI baseline
- environment template

Exit: clean checkout can install, build, test, and run core services locally.

### Week 2: Identity and Nepal localization
- user schema
- session/auth foundation
- Google OAuth
- Nepal +977 normalization
- OTP provider abstraction and abuse controls
- EN/NE translation architecture
- province/district/municipality/ward reference model

Exit: user can authenticate and select locale; core Nepal location data is usable.

### Week 3: Candidate/company domain
- candidate profile
- experience/education/skills
- companies
- company membership/roles
- employer authorization tests

Exit: candidate and employer identities/resources are securely separated.

## Phase 2 — Job Posting & Applying (Weeks 4-6)

### Week 4: Employer jobs
- create/edit/publish/close jobs
- validation
- employer dashboard foundation
- public SEO-friendly job pages
- structured metadata

### Week 5: Search/discovery
- job listing/search/filtering
- location/category/employment filters
- pagination
- indexing/query optimization
- saved jobs if schedule allows

### Week 6: Applications and resumes
- private resume upload
- PDF/DOCX validation
- SHA-256 fingerprinting
- local extraction/cache foundation
- application submit
- application status history
- basic ATS candidate list

Exit: complete non-AI candidate -> job -> employer application workflow.

## Phase 3 — Async Queue & AI Caching (Weeks 7-9)

### Week 7: Durable async pipeline
- BullMQ
- transactional outbox
- durable AI job table
- worker process
- retries/backoff/dead-letter handling
- worker observability

### Week 8: AI matching
- provider abstraction
- prompt/matching versioning
- strict structured schema
- score/pros/cons persistence
- JSONB + indexed score
- result cache
- token/cost telemetry

### Week 9: ATS integration and hardening
- ATS sorting/filtering by score/status
- match explanation UI
- manual/human-controlled workflow
- AI outage behavior
- load/performance tests for queue and ATS queries

Exit: application submit remains fast while AI results appear asynchronously.

## Phase 4 — Production Gateways & SEO Launch (Weeks 10-12)

### Week 10: eSewa and subscriptions
- plans/entitlements
- payment initiation
- eSewa UAT
- response/status verification
- idempotency
- reconciliation
- payment/admin views

### Week 11: Admin, moderation, security
- admin portal essentials
- user/company/job moderation
- AI cost dashboard
- audit logging
- security tests
- rate-limit tuning
- backup/restore test

### Week 12: Launch stabilization
- SEO review
- accessibility/responsive QA
- production deployment
- domain/HTTPS
- monitoring/alerts
- eSewa production readiness
- SMS production readiness
- smoke tests
- bug fixing
- project documentation/demo

Exit: controlled production MVP.

## Scope protection

If schedule pressure occurs, defer optional features before compromising authentication, authorization, payment verification, data integrity, resume privacy, background-job reliability, or tests for critical flows.

## Weekly operating rhythm

- Start week: choose a small issue set with acceptance criteria.
- During week: use feature branches and focused commits.
- Before merge: tests + review + docs where behavior changed.
- End week: demo working software and update risks/next priorities.

## MVP completion definition

The project is not finished merely because screens exist. Completion means candidate/employer/admin critical paths function end-to-end, AI is asynchronous/recoverable, eSewa entitlements are verified/idempotent, security boundaries are tested, production is observable/backed up, and the repository documents how to run and maintain the system.
