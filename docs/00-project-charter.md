# Project Charter

## Project
Nepal AI Job Platform

## Purpose
Create a Nepal-focused digital employment marketplace connecting candidates and employers, with AI-assisted resume/job matching, localized geographic data, bilingual UX, employer recruitment workflows, and Nepal-relevant payments.

## Problem
Job seekers often need to search across fragmented sources and repeatedly provide similar information. Employers must manually inspect large numbers of applications and have limited structured tools for ranking and managing candidates. Generic international products also frequently lack Nepal-specific location, phone, language, and payment support.

## MVP objectives

1. Let candidates create accounts, profiles, resumes, search jobs, apply, and track applications.
2. Let employers create company profiles, publish jobs, and manage applicants.
3. Process uploaded resumes safely and asynchronously.
4. Generate useful, explainable AI-assisted candidate/job matching without making autonomous hiring decisions.
5. Support English/Nepali UI and Nepal administrative geography.
6. Support relevant paid employer features using eSewa.
7. Give administrators moderation and operational controls.
8. Establish a secure and testable foundation that can evolve after MVP.

## Non-goals for the first MVP

- Fully autonomous candidate rejection or hiring
- AI-generated hiring decisions without human review
- A native iOS/Android application
- Payroll/HRIS functionality
- Video interviewing infrastructure
- Scraping third-party job sites without explicit permission
- Building our own large language model

## Primary actors

### Candidate
Owns a candidate profile, resumes, applications, preferences, and saved jobs.

### Employer user
Acts on behalf of an organization subject to role-based permissions.

### Administrator
Operates moderation, support, reference data, payment oversight, and platform management.

### Background worker
Processes expensive/non-interactive tasks such as resume parsing and AI matching.

## Success criteria for MVP

- A candidate can register, complete a profile, upload a supported resume, discover a job, and apply.
- An employer can register, configure a company, create a job, receive applications, and move candidates through statuses.
- Resume processing does not block normal web requests.
- Duplicate resume content can reuse cached processing through a cryptographic fingerprint.
- AI matching returns a bounded score plus explainable strengths/gaps and can fail without breaking applications.
- Nepal location and bilingual UX work across critical flows.
- Payment operations are verified server-side before paid entitlements are granted.
- Admins can moderate critical platform entities.
- Critical authorization and payment flows have automated tests.

## Constraints

- Initial delivery target: approximately 12 weeks.
- Keep the architecture understandable and operable by a small engineering team.
- Favor established libraries and managed infrastructure over unnecessary custom infrastructure.
- AI provider details should be replaceable behind an internal service boundary.

## Product principles

- Human-controlled hiring: AI informs; employers decide.
- Explainability over mysterious rankings.
- Privacy by default for candidate information.
- Nepal-first localization.
- Fast interactive paths; expensive work happens asynchronously.
- Secure server-side verification for identity, authorization, and money movement.

## Definition of done

A feature is done only when implementation, validation, authorization, tests, error handling, documentation, and relevant observability are included—not merely when the happy-path UI works.
