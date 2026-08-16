# Database Design Blueprint

PostgreSQL is the system of record. Redis is used only for ephemeral queue/cache coordination.

## Core integrity rules

- UUID primary keys for business entities.
- Timestamps use `TIMESTAMPTZ`.
- Foreign keys are explicit; destructive cascades are limited to true ownership relationships.
- Email/phone uniqueness uses partial unique indexes.
- Candidate application uniqueness is enforced in the database.
- Resume parse cache is keyed by a unique SHA-256 fingerprint.
- AI match score is projected into an indexed scalar column for fast ATS ordering; rich analysis remains JSONB.
- Money uses `NUMERIC`, never floating point.

## Core DDL

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE TYPE user_role AS ENUM ('JOB_SEEKER','EMPLOYER','ADMIN','SUPER_ADMIN');
CREATE TYPE application_status AS ENUM (
  'SUBMITTED','AI_PROCESSING','AI_COMPLETED','REVIEWING','SHORTLISTED',
  'INTERVIEW','OFFER','HIRED','REJECTED','WITHDRAWN','AI_FAILED'
);

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  role user_role NOT NULL DEFAULT 'JOB_SEEKER',
  phone_e164 VARCHAR(20),
  email VARCHAR(320),
  preferred_locale VARCHAR(2) NOT NULL DEFAULT 'en'
    CHECK (preferred_locale IN ('en','ne')),
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CHECK (phone_e164 IS NOT NULL OR email IS NOT NULL)
);

CREATE UNIQUE INDEX uq_users_phone ON users(phone_e164)
WHERE phone_e164 IS NOT NULL;
CREATE UNIQUE INDEX uq_users_email ON users(LOWER(email))
WHERE email IS NOT NULL;

CREATE TABLE provinces (
  id SMALLINT PRIMARY KEY CHECK (id BETWEEN 1 AND 7),
  name_en VARCHAR(80) NOT NULL UNIQUE,
  name_ne VARCHAR(120) NOT NULL
);

CREATE TABLE districts (
  id SMALLSERIAL PRIMARY KEY,
  province_id SMALLINT NOT NULL REFERENCES provinces(id),
  name_en VARCHAR(100) NOT NULL,
  name_ne VARCHAR(150),
  UNIQUE(province_id,name_en)
);

CREATE TABLE municipalities (
  id SERIAL PRIMARY KEY,
  district_id SMALLINT NOT NULL REFERENCES districts(id),
  name_en VARCHAR(150) NOT NULL,
  name_ne VARCHAR(200),
  municipality_type VARCHAR(40) NOT NULL,
  UNIQUE(district_id,name_en)
);

CREATE TABLE companies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug VARCHAR(160) NOT NULL UNIQUE,
  name_en VARCHAR(200) NOT NULL,
  name_ne VARCHAR(250),
  province_id SMALLINT REFERENCES provinces(id),
  district_id SMALLINT REFERENCES districts(id),
  municipality_id INTEGER REFERENCES municipalities(id),
  ward_no SMALLINT CHECK (ward_no IS NULL OR ward_no BETWEEN 1 AND 99),
  locality TEXT,
  is_verified BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE company_members (
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  membership_role VARCHAR(30) NOT NULL,
  PRIMARY KEY(company_id,user_id)
);

CREATE TABLE job_posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE RESTRICT,
  created_by UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  slug VARCHAR(220) NOT NULL UNIQUE,
  title_en VARCHAR(220) NOT NULL,
  title_ne VARCHAR(300),
  description_en TEXT NOT NULL,
  description_ne TEXT,
  requirements_en TEXT,
  requirements_ne TEXT,
  province_id SMALLINT REFERENCES provinces(id),
  district_id SMALLINT REFERENCES districts(id),
  municipality_id INTEGER REFERENCES municipalities(id),
  salary_min NUMERIC(14,2),
  salary_max NUMERIC(14,2),
  currency CHAR(3) NOT NULL DEFAULT 'NPR',
  status VARCHAR(30) NOT NULL DEFAULT 'DRAFT',
  published_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CHECK (salary_min IS NULL OR salary_max IS NULL OR salary_max >= salary_min)
);

CREATE INDEX idx_jobs_public ON job_posts(status,published_at DESC);
CREATE INDEX idx_jobs_location ON job_posts(province_id,district_id,municipality_id);

CREATE TABLE resumes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  object_key TEXT NOT NULL,
  original_filename TEXT NOT NULL,
  mime_type VARCHAR(120) NOT NULL,
  file_size_bytes BIGINT NOT NULL CHECK(file_size_bytes > 0),
  sha256 CHAR(64) NOT NULL,
  is_primary BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_resumes_user ON resumes(user_id,created_at DESC);

CREATE TABLE parsed_resumes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sha256 CHAR(64) NOT NULL UNIQUE,
  extracted_text TEXT NOT NULL CHECK(char_length(extracted_text) > 0),
  extractor VARCHAR(80) NOT NULL,
  extractor_version VARCHAR(40),
  extraction_metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE resume_parse_links (
  resume_id UUID PRIMARY KEY REFERENCES resumes(id) ON DELETE CASCADE,
  parsed_resume_id UUID NOT NULL REFERENCES parsed_resumes(id) ON DELETE RESTRICT
);

CREATE TABLE job_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_post_id UUID NOT NULL REFERENCES job_posts(id) ON DELETE RESTRICT,
  candidate_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  resume_id UUID NOT NULL REFERENCES resumes(id) ON DELETE RESTRICT,
  parsed_resume_id UUID NOT NULL REFERENCES parsed_resumes(id) ON DELETE RESTRICT,
  status application_status NOT NULL DEFAULT 'SUBMITTED',
  cover_letter TEXT,
  ai_evaluation JSONB,
  ai_match_score SMALLINT GENERATED ALWAYS AS (
    CASE WHEN ai_evaluation ? 'match_score'
      THEN (ai_evaluation->>'match_score')::SMALLINT
      ELSE NULL END
  ) STORED,
  ai_model VARCHAR(100),
  ai_prompt_version VARCHAR(50),
  ai_completed_at TIMESTAMPTZ,
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(job_post_id,candidate_id),
  CHECK(ai_match_score IS NULL OR ai_match_score BETWEEN 0 AND 100)
);

CREATE INDEX idx_app_job_score
ON job_applications(job_post_id,ai_match_score DESC)
WHERE ai_match_score IS NOT NULL;

CREATE INDEX idx_app_ai_jsonb
ON job_applications USING GIN(ai_evaluation jsonb_path_ops);
```

## Additional tables to implement

The implementation phase will add normalized tables for candidate profiles, experience, education, skills, job skills, application status history, saved jobs, OTP challenges, auth identities/sessions, plans, subscriptions, eSewa payments, AI jobs/usage ledger, transactional outbox, notifications, moderation reports, and audit logs.

## Migration policy

- Never manually mutate production schema.
- Every schema change is a version-controlled migration.
- Destructive migrations require an explicit data migration/rollback plan.
- Seed/reference data is separate from user-generated production data.
- Indexes must be justified by actual query paths and measured as the dataset grows.
