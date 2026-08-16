# Nepal AI Job Platform

An AI-powered job search and recruitment platform designed for Nepal.

> Status: Planning and architecture phase. We intentionally document the system before implementing production code.

## Vision

Build a trustworthy Nepal-focused employment platform that helps candidates discover relevant jobs and helps employers find suitable applicants using structured data and asynchronous AI-assisted matching.

## Core users

### Candidates
- Create and maintain a professional profile
- Upload PDF/DOCX resumes
- Search and filter jobs
- Apply to jobs
- Track application status
- Receive AI-assisted job match insights
- Use English or Nepali interfaces

### Employers
- Create and manage company profiles
- Publish and manage vacancies
- Review applicants through an ATS-style workflow
- View structured resume information and AI match insights
- Manage paid platform features through eSewa

### Administrators
- Moderate users, employers, companies, jobs, and platform content
- Review platform activity and payments
- Manage Nepal-specific reference data
- Monitor system health and AI-processing operations

## Planned architecture

This repository is planned as a monorepo with three primary applications:

- `apps/web` — candidate, employer, and admin web interfaces
- `apps/api` — core backend/API
- `apps/ai-worker` — asynchronous resume processing and AI matching

Shared capabilities will live under `packages/`, while infrastructure and migrations will be maintained separately.

## Major capabilities

- Candidate Portal
- Employer Dashboard / ATS
- Super Admin Portal
- Job discovery and applications
- Google authentication
- Nepal `+977` phone/OTP authentication
- English and Nepali localization
- Nepal province/district/municipality/ward support
- PDF and DOCX resume processing
- SHA-256 resume fingerprinting and result caching
- Redis-backed asynchronous job queues
- AI job/resume match score with explainable strengths and gaps
- PostgreSQL persistence with JSONB where appropriate
- eSewa payment integration
- Audit logging, authorization, validation, rate limiting, and secure secret handling

## Engineering principles

1. Document important decisions before implementation.
2. Keep AI processing asynchronous and outside request/response paths.
3. Never commit secrets or production credentials.
4. Treat authorization as a backend responsibility, not a UI feature.
5. Validate all external input.
6. Keep payment and authentication integrations isolated behind clear interfaces.
7. Prefer deterministic application logic; use AI only where it adds meaningful value.
8. Store enough metadata to audit important automated decisions.
9. Design Nepal localization as a core domain requirement rather than an afterthought.
10. Build and ship incrementally through reviewed GitHub changes.

## Documentation

Project documentation lives in [`docs/`](docs/):

- Project charter and scope
- Product requirements
- System architecture
- Database design
- API conventions
- AI/resume pipeline
- Payment architecture
- Security requirements
- Testing strategy
- Deployment strategy
- 12-week implementation roadmap

## Proposed repository structure

```text
.
├── apps/
│   ├── web/
│   ├── api/
│   └── ai-worker/
├── packages/
│   ├── database/
│   ├── auth/
│   ├── validation/
│   ├── localization/
│   └── shared/
├── docs/
├── infrastructure/
│   ├── docker/
│   └── migrations/
└── .github/
    ├── ISSUE_TEMPLATE/
    └── workflows/
```

Directories will be created as implementation begins; Git does not track empty directories.

## Delivery approach

The initial target is a 12-week MVP program:

1. Foundation and architecture
2. Authentication and user profiles
3. Employer/company and job management
4. Job search and applications
5. Resume ingestion and processing
6. AI matching pipeline
7. Employer ATS workflow
8. eSewa payments
9. Admin and moderation
10. Security, testing, observability, deployment, and launch stabilization

See [`docs/10-roadmap.md`](docs/10-roadmap.md) for the detailed plan.

## Current phase

**Phase 0 — Project definition.** No production feature should be considered complete until its requirements, security implications, data model, tests, and acceptance criteria are understood.

## License

No open-source license has been selected yet. Until one is deliberately added, the repository should be treated as private/proprietary project work.
