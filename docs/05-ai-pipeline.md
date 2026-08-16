# Cost-Optimized Resume and AI Pipeline

## Non-negotiable invariant

The platform must never send the same raw uploaded resume file to an LLM repeatedly. Raw file parsing is local; AI receives only the minimum normalized text necessary for the matching task.

## Resume cache pipeline

```text
Upload PDF/DOCX
      |
      v
stream file + server SHA-256
      |
      v
parsed_resumes lookup by SHA-256
   /                    \
HIT                     MISS
 |                        |
reuse cached text     local extraction
 |                        |
 |                   validate/normalize
 |                        |
 |                   INSERT cache row
  \______________________/
             |
             v
       parsed resume ready
```

### File rules

- Enforce allowlisted MIME/type and size limits.
- Verify content signatures where practical; do not trust filename extension alone.
- Store originals privately.
- Calculate SHA-256 on the server while streaming to avoid unnecessary memory duplication.
- Use PDF/DOCX extraction libraries locally.
- Treat OCR as a controlled fallback for scanned PDFs rather than the default path.

## Cache identity

Extraction cache identity begins with SHA-256 of exact uploaded bytes. Extraction metadata records extractor/version. If extraction logic changes incompatibly, the system must support reprocessing/versioning rather than silently treating old extraction as equivalent.

AI result cache key should include at least:

```text
resume_sha256
+ canonical_job_content_hash
+ prompt_version
+ model_family_or_matching_version
```

Changing the job description or matching policy invalidates the relevant result naturally.

## Application matching

For each application, the worker uses:
- cached resume text
- target job description/requirements
- bounded matching instructions

It does not re-extract the resume for every application.

## Structured output

Expected contract:

```json
{
  "schema_version": 1,
  "match_score": 82,
  "pros": [
    "Strong Node.js backend experience",
    "PostgreSQL experience is demonstrated"
  ],
  "cons": [
    "Production Kubernetes experience is not demonstrated"
  ]
}
```

Rules:
- `match_score` integer 0-100
- bounded `pros`/`cons` arrays
- concise strings
- no invented credentials or experience
- schema validation before persistence

## Human decision boundary

AI matching is an advisory ranking aid. It must not autonomously:
- hire or reject a candidate
- change application stage based solely on the score
- determine compensation
- infer protected/sensitive attributes for ranking

Employer decisions remain human-controlled.

## Privacy minimization

Where not needed for matching, avoid sending personal identifiers such as phone number, email, exact home address, date of birth, photographs, or other irrelevant PII to the model provider. Matching should focus on professional qualifications and job requirements.

## AI usage ledger

Track per transaction:
- provider/model
- prompt/matching version
- application/company identifiers
- input tokens
- cached input tokens when available
- output tokens
- latency
- retry count
- success/failure category
- estimated foreign-currency cost
- cache hit/miss

This data powers the Super Admin cost dashboard and cost controls.

## Reliability

- Explicit provider timeouts
- Exponential backoff + jitter for transient errors
- Maximum retry count
- Dead-letter state
- Idempotent result writes
- No queue message contains full resume text
- Candidate application survives AI outage

## Cost controls

1. Local file extraction.
2. Resume extraction cache by SHA-256.
3. AI match cache by resume/job/prompt/model identity.
4. Prompt text kept stable where provider prompt caching can benefit.
5. Bounded structured output.
6. No unnecessary re-evaluation on ATS page views.
7. Configurable daily/monthly budgets and alert thresholds.
8. Admin reporting by provider, model, company, feature, and time period.
