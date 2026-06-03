# Database Schema & Design
## AI Resume Analyzer

**Version**: 1.0  
**Database**: PostgreSQL 14+  
**ORM**: Prisma  
**Last Updated**: June 2026

---

## Table of Contents

1. [Entity Relationship Diagram](#entity-relationship-diagram)
2. [Table Definitions](#table-definitions)
3. [Indexes & Performance](#indexes--performance)
4. [Migrations](#migrations)
5. [Data Retention Policies](#data-retention-policies)
6. [Backup Strategy](#backup-strategy)

---

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USERS                                   │
├──────────────────────────────────────────────────────────────────┤
│ id (PK)                    │ UUID                                │
│ email (UNIQUE)             │ STRING                              │
│ password_hash              │ STRING                              │
│ first_name                 │ STRING                              │
│ last_name                  │ STRING                              │
│ user_type                  │ ENUM: job_seeker, recruiter, admin  │
│ subscription_tier          │ ENUM: free, pro, team, enterprise   │
│ created_at                 │ TIMESTAMP                           │
│ updated_at                 │ TIMESTAMP                           │
│ deleted_at (soft delete)   │ TIMESTAMP (nullable)                │
└─────────────────────────────────────────────────────────────────┘
          │
          ├──────────────────────────────────────────────────────────────┐
          │                                                              │
          ▼                                                              ▼
┌──────────────────────────────┐                         ┌──────────────────────────────┐
│      RESUMES                 │                         │    ANALYSES                  │
├──────────────────────────────┤                         ├──────────────────────────────┤
│ id (PK)                      │                         │ id (PK)                      │
│ user_id (FK → users.id)      │◄────────────────────────│ resume_id (FK → resumes.id)  │
│ file_name                    │                         │ user_id (FK → users.id)      │
│ file_url (S3 path)           │                         │ analysis_type                │
│ file_type                    │                         │ quality_score (0-100)        │
│ file_size                    │                         │ professionalism_score        │
│ parsed_content (JSON)        │                         │ ats_score (0-100)            │
│ parsing_status               │                         │ feedback (JSON)              │
│ created_at                   │                         │ recommendations (JSON)       │
│ updated_at                   │                         │ created_at                   │
│ deleted_at                   │                         │ completed_at                 │
└──────────────────────────────┘                         └──────────────────────────────┘
          │                                                              ▲
          │                                                              │
          └──────────────────────────────────────┐                      │
                                                 │                      │
                              ┌──────────────────┴──────────────────────┘
                              │
                              ▼
                    ┌──────────────────────────────┐
                    │        MATCHES               │
                    ├──────────────────────────────┤
                    │ id (PK)                      │
                    │ resume_id (FK)               │
                    │ job_id (FK)                  │
                    │ user_id (FK)                 │
                    │ match_score (0-100)          │
                    │ skill_match_score            │
                    │ experience_match_score       │
                    │ keyword_match_score          │
                    │ gap_analysis (JSON)          │
                    │ recommendations (JSON)       │
                    │ created_at                   │
                    └──────────────────────────────┘
                              ▲
                              │
          ┌───────────────────┴───────────────────┐
          │                                       │
          ▼                                       ▼
┌──────────────────────────────┐      ┌──────────────────────────────┐
│        JOBS                  │      │    SKILL_GAPS                │
├──────────────────────────────┤      ├──────────────────────────────┤
│ id (PK)                      │      │ id (PK)                      │
│ user_id (FK → users.id)      │      │ resume_id (FK)               │
│ title                        │      │ target_role                  │
│ description (TEXT)           │      │ missing_skills (JSON)        │
│ requirements (JSON)          │      │ learning_recommendations     │
│ salary_range                 │      │ estimated_months_to_learn    │
│ created_at                   │      │ created_at                   │
└──────────────────────────────┘      └──────────────────────────────┘
          ▲
          │
          └──────────────────┬──────────────────┐
                             │                  │
          ┌──────────────────┴──────────────────┴──────┐
          │                                           │
          ▼                                           ▼
┌──────────────────────────────────────┐  ┌──────────────────────────────────────┐
│      ANALYSIS_BATCHES                │  │    SUBSCRIPTIONS                     │
├──────────────────────────────────────┤  ├──────────────────────────────────────┤
│ id (PK)                              │  │ id (PK)                              │
│ user_id (FK → users.id)              │  │ user_id (FK → users.id)              │
│ job_id (FK → jobs.id)                │  │ tier                                 │
│ file_name                            │  │ stripe_customer_id                   │
│ total_resumes                        │  │ stripe_subscription_id               │
│ processed_count                      │  │ current_period_start                 │
│ status: pending, processing, ...     │  │ current_period_end                   │
│ results_url (S3)                     │  │ status: active, canceled, past_due   │
│ created_at                           │  │ created_at                           │
│ completed_at                         │  │ updated_at                           │
└──────────────────────────────────────┘  └──────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────┐
│      BATCH_RESUMES                   │
├──────────────────────────────────────┤
│ id (PK)                              │
│ batch_id (FK)                        │
│ resume_id (FK → resumes.id)          │
│ rank                                 │
│ match_score (0-100)                  │
│ processing_status                    │
└──────────────────────────────────────┘
```

---

## Table Definitions

### USERS

Core user management table.

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  user_type VARCHAR(50) NOT NULL CHECK (user_type IN ('job_seeker', 'recruiter', 'admin')),
  subscription_tier VARCHAR(50) NOT NULL DEFAULT 'free' 
    CHECK (subscription_tier IN ('free', 'pro', 'team', 'enterprise')),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);
```

**Fields**:
- `id`: Unique identifier
- `email`: User email (case-insensitive in app layer)
- `password_hash`: bcrypt hashed password (never store plain text)
- `first_name`, `last_name`: User name
- `user_type`: Distinguishes job seeker from recruiter from admin
- `subscription_tier`: Maps to features available
- `created_at`, `updated_at`: Timestamps for auditing
- `deleted_at`: Soft delete for data retention

---

### RESUMES

Stores uploaded resume files and metadata.

```sql
CREATE TABLE resumes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  file_name VARCHAR(255) NOT NULL,
  file_url VARCHAR(512) NOT NULL,  -- S3 path
  file_type VARCHAR(10) NOT NULL CHECK (file_type IN ('pdf', 'docx', 'txt')),
  file_size INTEGER NOT NULL,  -- bytes
  parsed_content JSONB,  -- Parsed structured data
  parsing_status VARCHAR(50) NOT NULL DEFAULT 'pending'
    CHECK (parsing_status IN ('pending', 'parsing', 'completed', 'failed')),
  parsing_error TEXT,  -- Error message if failed
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP
);
```

**parsed_content Structure**:
```json
{
  "contact": {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+1-555-0100",
    "location": "New York, NY"
  },
  "summary": "Summary text...",
  "experience": [
    {
      "company": "Company Name",
      "title": "Job Title",
      "startDate": "2020-01",
      "endDate": "2021-12",
      "description": "Responsibilities and achievements..."
    }
  ],
  "education": [
    {
      "school": "University Name",
      "degree": "BS",
      "field": "Computer Science",
      "graduationYear": 2014
    }
  ],
  "skills": ["Python", "AWS", "Kubernetes"],
  "certifications": [
    {
      "name": "AWS Solutions Architect",
      "issuer": "Amazon",
      "date": "2021-06"
    }
  ]
}
```

---

### ANALYSES

Stores analysis results for resumes.

```sql
CREATE TABLE analyses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  resume_id UUID NOT NULL REFERENCES resumes(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  analysis_type VARCHAR(50) NOT NULL CHECK (analysis_type IN ('quality', 'ats', 'full')),
  quality_score INTEGER CHECK (quality_score BETWEEN 0 AND 100),
  professionalism_score INTEGER CHECK (professionalism_score BETWEEN 0 AND 100),
  ats_score INTEGER CHECK (ats_score BETWEEN 0 AND 100),
  feedback JSONB,  -- Detailed feedback structure
  recommendations JSONB,  -- Array of recommendations
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP
);
```

**Indexes**:
```sql
CREATE INDEX idx_analyses_resume_id ON analyses(resume_id);
CREATE INDEX idx_analyses_user_id ON analyses(user_id);
CREATE INDEX idx_analyses_created_at ON analyses(created_at DESC);
```

---

### JOBS

Stores job descriptions for matching.

```sql
CREATE TABLE jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  requirements JSONB,  -- Structured job requirements
  salary_range VARCHAR(100),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**requirements Structure**:
```json
{
  "requiredSkills": ["Python", "AWS", "Kubernetes"],
  "preferredSkills": ["Go", "Docker"],
  "minimumYears": 5,
  "preferredYears": 7,
  "educationRequired": "BS Computer Science",
  "certifications": ["AWS Solutions Architect"],
  "industries": ["Technology"],
  "experienceLevels": ["Senior", "Staff"]
}
```

---

### MATCHES

Stores job-resume matching results.

```sql
CREATE TABLE matches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  resume_id UUID NOT NULL REFERENCES resumes(id) ON DELETE CASCADE,
  job_id UUID NOT NULL REFERENCES jobs(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  match_score INTEGER NOT NULL CHECK (match_score BETWEEN 0 AND 100),
  skill_match_score INTEGER CHECK (skill_match_score BETWEEN 0 AND 100),
  experience_match_score INTEGER CHECK (experience_match_score BETWEEN 0 AND 100),
  keyword_match_score INTEGER CHECK (keyword_match_score BETWEEN 0 AND 100),
  gap_analysis JSONB,  -- Detailed gap breakdown
  recommendations JSONB,  -- Recommendations for improvement
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Indexes**:
```sql
CREATE INDEX idx_matches_resume_id ON matches(resume_id);
CREATE INDEX idx_matches_job_id ON matches(job_id);
CREATE INDEX idx_matches_user_id ON matches(user_id);
CREATE INDEX idx_matches_composite ON matches(user_id, job_id);
```

---

### SKILL_GAPS

Stores skill gap analysis results.

```sql
CREATE TABLE skill_gaps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  resume_id UUID NOT NULL REFERENCES resumes(id) ON DELETE CASCADE,
  target_role VARCHAR(255) NOT NULL,
  industry VARCHAR(100),
  missing_skills JSONB,  -- Array of missing skills with priority
  learning_recommendations JSONB,  -- Courses, certifications, paths
  estimated_months_to_learn INTEGER,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

### ANALYSIS_BATCHES

Stores recruiter batch processing jobs.

```sql
CREATE TABLE analysis_batches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  job_id UUID REFERENCES jobs(id) ON DELETE SET NULL,
  batch_name VARCHAR(255),
  file_name VARCHAR(255),
  total_resumes INTEGER NOT NULL,
  processed_count INTEGER NOT NULL DEFAULT 0,
  failed_count INTEGER NOT NULL DEFAULT 0,
  status VARCHAR(50) NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'queued', 'processing', 'completed', 'failed', 'cancelled')),
  results_url VARCHAR(512),  -- S3 path to results CSV/PDF
  error_message TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP
);
```

**Indexes**:
```sql
CREATE INDEX idx_batches_user_id ON analysis_batches(user_id);
CREATE INDEX idx_batches_status ON analysis_batches(status);
CREATE INDEX idx_batches_created_at ON analysis_batches(created_at DESC);
```

---

### BATCH_RESUMES

Maps individual resumes to batch processing jobs.

```sql
CREATE TABLE batch_resumes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  batch_id UUID NOT NULL REFERENCES analysis_batches(id) ON DELETE CASCADE,
  resume_id UUID NOT NULL REFERENCES resumes(id) ON DELETE CASCADE,
  rank INTEGER,
  match_score INTEGER CHECK (match_score BETWEEN 0 AND 100),
  processing_status VARCHAR(50) DEFAULT 'pending'
    CHECK (processing_status IN ('pending', 'processing', 'completed', 'failed')),
  error_message TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Indexes**:
```sql
CREATE INDEX idx_batch_resumes_batch_id ON batch_resumes(batch_id);
CREATE INDEX idx_batch_resumes_resume_id ON batch_resumes(resume_id);
CREATE INDEX idx_batch_resumes_rank ON batch_resumes(batch_id, rank);
```

---

### SUBSCRIPTIONS

Manages user subscription information.

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  tier VARCHAR(50) NOT NULL CHECK (tier IN ('free', 'pro', 'team', 'enterprise')),
  stripe_customer_id VARCHAR(255),
  stripe_subscription_id VARCHAR(255),
  current_period_start TIMESTAMP,
  current_period_end TIMESTAMP,
  status VARCHAR(50) NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'canceled', 'past_due', 'trialing')),
  cancel_at_period_end BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

---

## Indexes & Performance

### Primary Indexes (Required)

```sql
-- Foreign key lookup performance
CREATE INDEX idx_resumes_user_id ON resumes(user_id);
CREATE INDEX idx_analyses_resume_id ON analyses(resume_id);
CREATE INDEX idx_analyses_user_id ON analyses(user_id);
CREATE INDEX idx_matches_resume_id ON matches(resume_id);
CREATE INDEX idx_matches_job_id ON matches(job_id);
CREATE INDEX idx_jobs_user_id ON jobs(user_id);
CREATE INDEX idx_batches_user_id ON analysis_batches(user_id);
CREATE INDEX idx_batch_resumes_batch_id ON batch_resumes(batch_id);
```

### Search & Query Optimization

```sql
-- Time-based queries
CREATE INDEX idx_resumes_created_at ON resumes(created_at DESC);
CREATE INDEX idx_analyses_completed_at ON analyses(completed_at DESC);
CREATE INDEX idx_matches_created_at ON matches(created_at DESC);
CREATE INDEX idx_batches_created_at ON analysis_batches(created_at DESC);

-- Status filtering
CREATE INDEX idx_resumes_status ON resumes(parsing_status);
CREATE INDEX idx_batches_status ON analysis_batches(status);
CREATE INDEX idx_batch_resumes_status ON batch_resumes(processing_status);

-- Composite indexes for common queries
CREATE INDEX idx_matches_user_job ON matches(user_id, job_id);
CREATE INDEX idx_batch_resumes_batch_rank ON batch_resumes(batch_id, rank);
```

### JSONB Query Optimization

```sql
-- For parsed_content JSON queries
CREATE INDEX idx_resumes_content_gin ON resumes USING GIN(parsed_content);

-- For requirements JSON queries
CREATE INDEX idx_jobs_requirements_gin ON jobs USING GIN(requirements);
```

### Index Usage Examples

```sql
-- Fast lookup of user's resumes
SELECT * FROM resumes WHERE user_id = 'user-123' ORDER BY created_at DESC;

-- Fast batch processing queries
SELECT * FROM batch_resumes 
WHERE batch_id = 'batch-123' 
ORDER BY rank 
LIMIT 20;

-- JSON search for skills
SELECT * FROM resumes 
WHERE parsed_content @> '{"skills": "Python"}'
AND user_id = 'user-123';
```

---

## Migrations

### Prisma Schema Setup

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                String   @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  email             String   @unique
  passwordHash      String
  firstName         String
  lastName          String
  userType          String   @db.VarChar(50)
  subscriptionTier  String   @db.VarChar(50) @default("free")
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
  deletedAt         DateTime?

  resumes          Resume[]
  analyses         Analysis[]
  matches          Match[]
  jobs             Job[]
  batches          AnalysisBatch[]
  subscription     Subscription?

  @@index([email])
  @@index([userType])
}

model Resume {
  id               String   @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  userId           String   @db.Uuid
  fileName         String
  fileUrl          String
  fileType         String   @db.VarChar(10)
  fileSize         Int
  parsedContent    Json?    @db.JsonB
  parsingStatus    String   @db.VarChar(50) @default("pending")
  parsingError     String?
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
  deletedAt        DateTime?

  user    User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  analyses       Analysis[]
  matches        Match[]
  skillGaps      SkillGap[]
  batchResumes   BatchResume[]

  @@index([userId])
  @@index([parsingStatus])
  @@index([createdAt])
}

// ... continue with other models
```

### Migration Commands

```bash
# Create database schema from Prisma schema
npm run prisma:migrate:dev -- --name initial_schema

# Run pending migrations (production)
npm run prisma:migrate:deploy

# Check migration status
npm run prisma:migrate:status

# Reset database (development only)
npm run prisma:migrate:reset
```

---

## Data Retention Policies

### Soft Deletes

All main tables support soft delete via `deleted_at` field:

```sql
-- Query active records only
SELECT * FROM resumes WHERE deleted_at IS NULL;

-- Hard delete after 90 days (GDPR compliance)
DELETE FROM resumes WHERE deleted_at < NOW() - INTERVAL '90 days';
```

### S3 File Cleanup

```python
# Cleanup old resume files (scheduled job)
import boto3
from datetime import datetime, timedelta

s3 = boto3.client('s3')

# Delete files for deleted resumes older than 90 days
def cleanup_s3_files():
    deleted_before = datetime.now() - timedelta(days=90)
    
    resumes = db.query(Resume)\
        .filter(Resume.deleted_at < deleted_before)\
        .all()
    
    for resume in resumes:
        s3.delete_object(Bucket='resumeanalyzer', Key=resume.file_url)
```

### Cache Cleanup

```bash
# Clear old analysis results from Redis (daily job)
# Keep results for 60 days
redis-cli EVAL "
  local keys = redis.call('keys', 'analysis:*')
  for i,key in ipairs(keys) do
    local ttl = redis.call('ttl', key)
    if ttl == -1 then
      redis.call('expire', key, 60*60*24*60)
    end
  end
" 0
```

---

## Backup Strategy

### PostgreSQL Backups

```sql
-- Automated daily backups using AWS RDS
-- - Full backup daily at 2 AM UTC
-- - Automated 30-day retention
-- - Multi-AZ deployment for redundancy

-- Manual backup before major deployments
pg_dump resumeanalyzer_db > backup_$(date +%Y%m%d_%H%M%S).sql

-- Restore from backup
psql resumeanalyzer_db < backup_20260603_000000.sql
```

### Point-in-Time Recovery

```bash
# Restore to specific point in time
# AWS RDS provides automated WAL (Write-Ahead Logs)
# Can restore to any point within 7 days

aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier resumeanalyzer-prod \
  --db-instance-identifier resumeanalyzer-restored \
  --restore-time 2026-06-02T10:00:00Z
```

### S3 Backup

```bash
# Cross-region replication for resume files
aws s3api put-bucket-replication \
  --bucket resumeanalyzer-files \
  --replication-configuration '{
    "Role": "arn:aws:iam::123456789012:role/s3-replication",
    "Rules": [{
      "Status": "Enabled",
      "Priority": 1,
      "DeleteMarkerReplication": {"Status": "Enabled"},
      "Filter": {"Prefix": ""},
      "Destination": {
        "Bucket": "arn:aws:s3:::resumeanalyzer-files-backup",
        "ReplicationTime": {"Status": "Enabled", "Time": {"Minutes": 15}}
      }
    }]
  }'
```

### Recovery Testing

```bash
# Monthly recovery drills
# 1. Restore database to staging
# 2. Restore S3 files from backup
# 3. Verify data integrity
# 4. Document recovery time
```

---

## Query Patterns

### Common Queries

```sql
-- Get user's resumes with latest analysis
SELECT r.*, a.quality_score, a.ats_score
FROM resumes r
LEFT JOIN analyses a ON r.id = a.resume_id AND a.analysis_type = 'full'
WHERE r.user_id = $1 AND r.deleted_at IS NULL
ORDER BY r.created_at DESC;

-- Get top matching candidates for batch
SELECT br.rank, br.match_score, r.*, b.batch_id
FROM batch_resumes br
JOIN resumes r ON br.resume_id = r.id
WHERE br.batch_id = $1
ORDER BY br.rank
LIMIT 20;

-- Get user subscription status with tier
SELECT u.*, s.tier, s.current_period_end, s.status
FROM users u
LEFT JOIN subscriptions s ON u.id = s.user_id
WHERE u.id = $1;
```

---

## Monitoring & Alerts

### Key Metrics

```sql
-- Monitor table sizes
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) 
FROM pg_tables
WHERE schemaname='public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Monitor index usage
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Monitor slow queries
SELECT 
  query,
  mean_exec_time,
  max_exec_time,
  calls
FROM pg_stat_statements
WHERE mean_exec_time > 1000  -- > 1 second
ORDER BY mean_exec_time DESC;
```

### Query Performance Targets

- Insert: < 100ms
- Select single record: < 50ms
- Select with join (2-3 tables): < 200ms
- Full table scan (small): < 1s
