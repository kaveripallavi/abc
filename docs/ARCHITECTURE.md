# Architecture Design Document
## AI Resume Analyzer

**Version**: 1.0  
**Status**: Planning  
**Last Updated**: June 2026

---

## 1. System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer                             │
├─────────────────────────────────────────────────────────────┤
│  Web UI (React/TypeScript)  │  Mobile (React Native)         │
│  Vite                       │  (Phase 3)                     │
├─────────────────────────────────────────────────────────────┤
│                     CDN (CloudFlare)                         │
├─────────────────────────────────────────────────────────────┤
│                     API Gateway                              │
│              (Express with rate limiting)                    │
├─────────────────────────────────────────────────────────────┤
│                  Backend Services Layer                       │
├──────────────┬──────────────┬──────────────┬────────────────┤
│   Auth       │   Resume     │   Analysis   │    Job Match   │
│   Service    │   Service    │   Service    │    Service     │
├──────────────┼──────────────┼──────────────┼────────────────┤
│ PostgreSQL   │   S3/Blob    │  Redis Cache │   Vector DB    │
│ (user data)  │  (files)     │ (results)    │  (embeddings)  │
├──────────────┴──────────────┴──────────────┴────────────────┤
│              Message Queue (Bull/Redis)                      │
├──────────────────────────────────────────────────────────────┤
│         AI Processing Layer (Background Workers)             │
├──────────────┬──────────────┬──────────────────────────────┤
│ Parser       │ LLM          │ Skill Matcher                 │
│ Service      │ Service      │ Service                       │
├──────────────┼──────────────┼──────────────────────────────┤
│ pdfparse,    │ OpenAI/      │ Embeddings +                 │
│ docx,        │ Claude       │ Similarity Search            │
│ textract     │ APIs         │                              │
├──────────────┴──────────────┴──────────────────────────────┤
│        External Services (3rd Party APIs)                    │
├──────────────┬──────────────┬──────────────────────────────┤
│ LLM APIs     │ Payment      │ Email Service                 │
│ (OpenAI)     │ (Stripe)     │ (SendGrid)                    │
└──────────────┴──────────────┴──────────────────────────────┘
```

---

## 2. Technology Stack

### 2.1 Frontend
- **Framework**: React 18+ with TypeScript
- **Build Tool**: Vite
- **State Management**: Redux Toolkit or Zustand
- **UI Components**: shadcn/ui with Tailwind CSS
- **HTTP Client**: Axios or TanStack Query
- **Form Handling**: React Hook Form with Zod validation
- **File Upload**: React Dropzone
- **Document Viewer**: React PDF, Docx Viewer
- **Charts**: Recharts for analytics
- **Testing**: Vitest, React Testing Library

### 2.2 Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js
- **Language**: TypeScript
- **Database ORM**: Prisma
- **Validation**: Zod or Joi
- **Authentication**: JWT + refresh tokens
- **API Documentation**: Swagger/OpenAPI
- **Logging**: Winston or Pino
- **Monitoring**: Datadog or New Relic
- **Testing**: Jest, Supertest

### 2.3 Infrastructure
- **Hosting**: AWS (or similar cloud provider)
  - **Compute**: ECS/EKS with auto-scaling
  - **Database**: RDS PostgreSQL
  - **Cache**: ElastiCache Redis
  - **Storage**: S3 for file uploads
  - **Queue**: SQS or managed Redis
  - **CDN**: CloudFront
- **Infrastructure as Code**: Terraform
- **CI/CD**: GitHub Actions
- **Containerization**: Docker

### 2.4 AI/ML Services
- **LLM Provider**: OpenAI GPT-4 or Claude 3
- **Embeddings**: OpenAI embeddings or similar
- **Vector Store**: Pinecone or Weaviate
- **NLP Libraries**: Natural, NLTK-equivalent JavaScript

---

## 3. Folder Structure

```
xyz/
├── docs/                          # Documentation
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── TASKS.md
│   ├── DECISIONS.md
│   ├── API.md
│   └── DATABASE.md
│
├── backend/                       # Node.js/Express Backend
│   ├── src/
│   │   ├── config/               # Configuration management
│   │   │   ├── env.ts
│   │   │   ├── database.ts
│   │   │   └── llm.ts
│   │   │
│   │   ├── middleware/           # Express middleware
│   │   │   ├── auth.ts
│   │   │   ├── errorHandler.ts
│   │   │   ├── validation.ts
│   │   │   └── rateLimit.ts
│   │   │
│   │   ├── routes/               # API routes
│   │   │   ├── auth.ts
│   │   │   ├── resumes.ts
│   │   │   ├── analysis.ts
│   │   │   ├── jobs.ts
│   │   │   └── admin.ts
│   │   │
│   │   ├── services/             # Business logic
│   │   │   ├── auth.service.ts
│   │   │   ├── resume.service.ts
│   │   │   ├── parser.service.ts
│   │   │   ├── analysis.service.ts
│   │   │   ├── jobMatch.service.ts
│   │   │   ├── skillGap.service.ts
│   │   │   └── export.service.ts
│   │   │
│   │   ├── workers/              # Background job processors
│   │   │   ├── parseResume.ts
│   │   │   ├── analyzeResume.ts
│   │   │   ├── matchJob.ts
│   │   │   └── generateReport.ts
│   │   │
│   │   ├── models/               # Database models (Prisma schema)
│   │   │   └── schema.prisma
│   │   │
│   │   ├── types/                # TypeScript types & interfaces
│   │   │   ├── resume.ts
│   │   │   ├── analysis.ts
│   │   │   ├── user.ts
│   │   │   └── common.ts
│   │   │
│   │   ├── utils/                # Utility functions
│   │   │   ├── validators.ts
│   │   │   ├── transformers.ts
│   │   │   ├── llm.ts            # LLM API wrappers
│   │   │   ├── file.ts           # File handling utilities
│   │   │   └── logger.ts
│   │   │
│   │   ├── lib/                  # External integrations
│   │   │   ├── storage.ts        # S3/Blob storage
│   │   │   ├── cache.ts          # Redis cache
│   │   │   ├── queue.ts          # Job queue (Bull)
│   │   │   ├── payment.ts        # Stripe integration
│   │   │   └── email.ts          # SendGrid integration
│   │   │
│   │   └── server.ts             # Express app initialization
│   │
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── fixtures/
│   │
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   │
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env.example
│   ├── .eslintrc.json
│   ├── jest.config.js
│   └── Dockerfile
│
├── frontend/                      # React/Vite Frontend
│   ├── src/
│   │   ├── components/           # Reusable React components
│   │   │   ├── common/
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   ├── Footer.tsx
│   │   │   │   └── LoadingSpinner.tsx
│   │   │   │
│   │   │   ├── auth/
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   ├── SignupForm.tsx
│   │   │   │   └── ProtectedRoute.tsx
│   │   │   │
│   │   │   ├── resume/
│   │   │   │   ├── ResumeUpload.tsx
│   │   │   │   ├── ResumePreview.tsx
│   │   │   │   ├── ResumeList.tsx
│   │   │   │   └── ResumeEditor.tsx
│   │   │   │
│   │   │   ├── analysis/
│   │   │   │   ├── QualityScore.tsx
│   │   │   │   ├── FeedbackPanel.tsx
│   │   │   │   ├── RecommendationCard.tsx
│   │   │   │   └── AnalysisReport.tsx
│   │   │   │
│   │   │   ├── matching/
│   │   │   │   ├── JobInput.tsx
│   │   │   │   ├── MatchScore.tsx
│   │   │   │   ├── SkillMatrix.tsx
│   │   │   │   └── GapAnalysis.tsx
│   │   │   │
│   │   │   └── recruiter/
│   │   │       ├── BulkUpload.tsx
│   │   │       ├── CandidateList.tsx
│   │   │       ├── RankingTable.tsx
│   │   │       └── ComparisonView.tsx
│   │   │
│   │   ├── pages/                # Page components
│   │   │   ├── Dashboard.tsx
│   │   │   ├── UploadResume.tsx
│   │   │   ├── AnalyzeResume.tsx
│   │   │   ├── MatchJob.tsx
│   │   │   ├── Skills.tsx
│   │   │   ├── RecruiterDashboard.tsx
│   │   │   ├── Pricing.tsx
│   │   │   └── Settings.tsx
│   │   │
│   │   ├── hooks/                # Custom React hooks
│   │   │   ├── useAuth.ts
│   │   │   ├── useResume.ts
│   │   │   ├── useAnalysis.ts
│   │   │   └── useApi.ts
│   │   │
│   │   ├── services/             # API service layer
│   │   │   ├── api.ts            # Axios instance
│   │   │   ├── auth.api.ts
│   │   │   ├── resume.api.ts
│   │   │   ├── analysis.api.ts
│   │   │   └── recruiter.api.ts
│   │   │
│   │   ├── store/                # Redux/Zustand store
│   │   │   ├── authSlice.ts
│   │   │   ├── resumeSlice.ts
│   │   │   ├── analysisSlice.ts
│   │   │   └── store.ts
│   │   │
│   │   ├── types/                # TypeScript types
│   │   │   ├── resume.ts
│   │   │   ├── analysis.ts
│   │   │   ├── user.ts
│   │   │   └── common.ts
│   │   │
│   │   ├── utils/                # Utility functions
│   │   │   ├── formatters.ts
│   │   │   ├── validators.ts
│   │   │   └── constants.ts
│   │   │
│   │   ├── styles/               # Global styles
│   │   │   ├── globals.css
│   │   │   └── tailwind.config.js
│   │   │
│   │   ├── App.tsx
│   │   └── main.tsx
│   │
│   ├── tests/
│   │   ├── unit/
│   │   └── integration/
│   │
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── .env.example
│   ├── .eslintrc.json
│   ├── vitest.config.ts
│   └── Dockerfile
│
├── .github/
│   └── workflows/                # CI/CD pipelines
│       ├── test.yml
│       ├── build.yml
│       └── deploy.yml
│
├── .gitignore
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 4. API Design

### 4.1 API Base URL
```
https://api.resumeanalyzer.com/v1
```

### 4.2 Authentication
- **Method**: JWT Bearer tokens
- **Token Refresh**: Refresh token rotation
- **Endpoints**:
  - `POST /auth/register` - User registration
  - `POST /auth/login` - User login
  - `POST /auth/refresh` - Refresh access token
  - `POST /auth/logout` - Logout

### 4.3 Resume Endpoints
```
POST   /resumes/upload              # Upload resume
GET    /resumes                     # List user's resumes
GET    /resumes/:id                 # Get resume details
GET    /resumes/:id/content         # Get parsed content
DELETE /resumes/:id                 # Delete resume
PATCH  /resumes/:id                 # Update resume metadata
```

### 4.4 Analysis Endpoints
```
POST   /analysis/quality            # Analyze resume quality
POST   /analysis/ats-score          # Check ATS compatibility
POST   /analysis/:resumeId/full     # Full analysis
GET    /analysis/:resumeId          # Get analysis result
```

### 4.5 Job Matching Endpoints
```
POST   /matching/score              # Score resume against job
POST   /matching/compare            # Compare resume vs job description
GET    /matching/:matchId           # Get detailed match analysis
POST   /matching/:matchId/export    # Export match report
```

### 4.6 Skill Gap Endpoints
```
POST   /skills/gap-analysis         # Analyze skill gaps
GET    /skills/gap-analysis/:id     # Get gap analysis result
POST   /skills/recommendations      # Get learning recommendations
```

### 4.7 Recruiter Endpoints
```
POST   /recruiter/batch-upload      # Batch upload resumes
GET    /recruiter/batches           # List batch jobs
GET    /recruiter/batches/:id       # Get batch status
POST   /recruiter/batches/:id/rank  # Rank candidates
GET    /recruiter/batches/:id/export # Export results
```

### 4.8 Response Format
```json
{
  "success": true,
  "data": { /* response data */ },
  "error": null,
  "timestamp": "2026-06-03T10:00:00Z",
  "requestId": "req-123456"
}
```

Error response:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [{ "field": "email", "message": "Must be valid email" }]
  },
  "timestamp": "2026-06-03T10:00:00Z",
  "requestId": "req-123456"
}
```

---

## 5. Database Schema

### 5.1 Core Tables

**Users**
```sql
users {
  id: UUID (PK)
  email: STRING (UNIQUE)
  password_hash: STRING
  first_name: STRING
  last_name: STRING
  subscription_tier: ENUM(free, pro, team, enterprise)
  is_recruiter: BOOLEAN
  created_at: TIMESTAMP
  updated_at: TIMESTAMP
  deleted_at: TIMESTAMP (soft delete)
}
```

**Resumes**
```sql
resumes {
  id: UUID (PK)
  user_id: UUID (FK)
  file_name: STRING
  file_url: STRING (S3)
  file_type: ENUM(pdf, docx, txt)
  file_size: INTEGER
  parsed_content: JSON
  parsing_status: ENUM(pending, completed, failed)
  created_at: TIMESTAMP
  updated_at: TIMESTAMP
  deleted_at: TIMESTAMP
}
```

**Analyses**
```sql
analyses {
  id: UUID (PK)
  resume_id: UUID (FK)
  user_id: UUID (FK)
  analysis_type: ENUM(quality, ats, full)
  quality_score: INTEGER (0-100)
  professionalism_score: INTEGER (0-100)
  ats_score: INTEGER (0-100)
  feedback: JSON
  recommendations: JSON
  created_at: TIMESTAMP
  completed_at: TIMESTAMP
}
```

**Jobs**
```sql
jobs {
  id: UUID (PK)
  user_id: UUID (FK)
  title: STRING
  description: TEXT
  requirements: JSON
  salary_range: STRING
  created_at: TIMESTAMP
}
```

**Matches**
```sql
matches {
  id: UUID (PK)
  resume_id: UUID (FK)
  job_id: UUID (FK)
  user_id: UUID (FK)
  match_score: INTEGER (0-100)
  skill_match_score: INTEGER (0-100)
  experience_match_score: INTEGER (0-100)
  keyword_match_score: INTEGER (0-100)
  gap_analysis: JSON
  recommendations: JSON
  created_at: TIMESTAMP
}
```

**SkillGaps**
```sql
skill_gaps {
  id: UUID (PK)
  resume_id: UUID (FK)
  target_role: STRING
  missing_skills: JSON
  learning_recommendations: JSON
  estimated_months_to_learn: INTEGER
  created_at: TIMESTAMP
}
```

**AnalysisBatches** (Recruiter feature)
```sql
analysis_batches {
  id: UUID (PK)
  user_id: UUID (FK)
  job_id: UUID (FK)
  file_name: STRING
  total_resumes: INTEGER
  processed_count: INTEGER
  status: ENUM(pending, processing, completed, failed)
  results_url: STRING
  created_at: TIMESTAMP
  completed_at: TIMESTAMP
}
```

**BatchResumes**
```sql
batch_resumes {
  id: UUID (PK)
  batch_id: UUID (FK)
  resume_id: UUID (FK)
  rank: INTEGER
  match_score: INTEGER
  processing_status: ENUM(pending, completed, failed)
}
```

**Subscriptions**
```sql
subscriptions {
  id: UUID (PK)
  user_id: UUID (FK)
  tier: ENUM(free, pro, team, enterprise)
  stripe_customer_id: STRING
  stripe_subscription_id: STRING
  current_period_start: TIMESTAMP
  current_period_end: TIMESTAMP
  status: ENUM(active, canceled, past_due)
  created_at: TIMESTAMP
  updated_at: TIMESTAMP
}
```

### 5.2 Indexes
- `users(email)` - UNIQUE
- `resumes(user_id)` - Query by user
- `analyses(resume_id)` - Query analysis for resume
- `matches(resume_id, job_id)` - Query matches
- `matches(user_id)` - Query user's matches
- `analysis_batches(user_id)` - Query recruiter batches

---

## 6. AI Workflow

### 6.1 Resume Parsing Flow
```
1. File Upload
   ↓
2. Validate file (type, size)
   ↓
3. Store in S3
   ↓
4. Enqueue parse job
   ↓
5. Background Worker: Extract text
   - Use pdf-parse, docx, or raw text
   - OCR if image-based PDF
   ↓
6. Parse structured data
   - Contact info, education, experience, skills, certifications
   - Use regex, NLP, and rule-based extraction
   ↓
7. Run through LLM for validation/enrichment
   ↓
8. Store parsed content in DB
   ↓
9. Update resume status to "completed"
   ↓
10. Notify user
```

### 6.2 Quality Analysis Flow
```
1. Resume analysis requested
   ↓
2. Retrieve parsed resume content
   ↓
3. Calculate structural metrics:
   - Section presence (contact, summary, experience, education, skills)
   - Length consistency (ideal ranges)
   - Formatting consistency (bullet points, dates)
   ↓
4. LLM Analysis:
   - Clarity assessment
   - Professionalism evaluation
   - Action verb usage
   - Metric quantification
   ↓
5. Generate detailed feedback with section scores
   ↓
6. Create recommendations:
   - Specific improvements
   - Rewritten examples
   - Keywords to add
   ↓
7. Calculate overall quality score (weighted)
   ↓
8. Store and return results
```

### 6.3 Job Matching Flow
```
1. User provides job description + resume
   ↓
2. Parse job description:
   - Extract requirements, skills, experience level
   - Identify keywords, must-haves, nice-to-haves
   ↓
3. LLM embeddings:
   - Convert resume content to embeddings
   - Convert job requirements to embeddings
   - Store in vector DB
   ↓
4. Similarity matching:
   - Skill match (semantic + keyword)
   - Experience match (years, domains, levels)
   - Industry/domain match
   ↓
5. Generate gap analysis:
   - Missing hard skills
   - Missing soft skills
   - Experience gaps
   - Keywords missing from resume
   ↓
6. Produce recommendations:
   - Keywords to add
   - Experience to emphasize
   - Skills to highlight
   - Learning path for gaps
   ↓
7. Calculate composite match score
   ↓
8. Return detailed report
```

### 6.4 Skill Gap Analysis Flow
```
1. User specifies target role
   ↓
2. Retrieve job market data for role
   - From internal database or external API
   - Extract common skill requirements
   - Identify trending skills
   ↓
3. Extract current skills from resume
   ↓
4. LLM semantic matching:
   - Match resume skills to market requirements
   - Identify exact and close matches
   ↓
5. Calculate gap:
   - Missing must-have skills
   - Missing nice-to-have skills
   - Skill level gaps
   ↓
6. Generate learning recommendations:
   - Priority order (must-have first)
   - Time-to-proficiency estimates
   - Course/resource suggestions
   ↓
7. Store and return results
```

### 6.5 Batch Processing Flow (Recruiter)
```
1. Recruiter uploads CSV with resumes or resume files
   ↓
2. Enqueue batch processing job
   ↓
3. Background workers process in parallel:
   - Parse each resume
   - Run full analysis
   ↓
4. Job description processing:
   - Extract requirements from provided job
   ↓
5. Parallel matching:
   - Compare each resume to job description
   - Calculate match scores
   ↓
6. Ranking:
   - Sort candidates by composite score
   - Generate candidate comparison matrix
   ↓
7. Generate report:
   - Shortlist with rankings
   - Skill gap summary per candidate
   - Recommendations for each
   ↓
8. Store results and notify recruiter
```

---

## 7. Data Flow Diagrams

### 7.1 Resume Ingestion to Analysis
```
User Upload
    ↓
API Endpoint (POST /resumes/upload)
    ↓
Validate & Store → S3
    ↓
Enqueue Job (Bull Queue)
    ↓
Background Worker
    ↓
├─ Extract Text (pdf-parse/docx)
├─ Parse Structure (regex/NLP)
├─ LLM Enrichment
└─ Store in DB
    ↓
Analysis Engine (triggered automatically)
    ↓
├─ Calculate Scores
├─ Generate Feedback
└─ Create Recommendations
    ↓
Store Analysis Result
    ↓
User Notification
```

### 7.2 Job Matching Workflow
```
Resume + Job Description
    ↓
API Endpoint (POST /matching/score)
    ↓
Parallel Processing:
├─ Parse Job Requirements
├─ Extract Resume Skills
└─ Generate Embeddings
    ↓
Similarity Matching
    ↓
Gap Analysis
    ↓
Recommendation Generation
    ↓
Score Calculation & Ranking
    ↓
Store & Return Results
```

---

## 8. Caching Strategy

### 8.1 Redis Cache Layers
- **User Sessions**: TTL 24h
- **Resume Parsed Content**: TTL 30 days
- **Analysis Results**: TTL 60 days (invalidate on resume update)
- **Job Requirements**: TTL 7 days
- **Embeddings**: TTL permanent (stored in vector DB)
- **Skill Taxonomy**: TTL 30 days

### 8.2 Cache Invalidation
- On resume update/re-analysis: invalidate analysis cache
- On job update: invalidate all matches for that job
- User logout: invalidate session

---

## 9. Security Architecture

### 9.1 Authentication & Authorization
- JWT-based authentication with refresh token rotation
- Role-based access control: User, Recruiter, Admin
- Subscription tier gating for features

### 9.2 Data Protection
- Encryption at rest: AES-256 in S3
- Encryption in transit: TLS 1.3
- PII field encryption in database
- Audit logging for sensitive operations

### 9.3 API Security
- Rate limiting: 100 req/min per user
- CORS: Whitelist frontend domain
- Input validation: Zod on all endpoints
- SQL injection prevention: Parameterized queries (Prisma)
- CSRF: CSRF tokens for state-changing operations

---

## 10. Scalability & Performance

### 10.1 Horizontal Scaling
- **API Servers**: Stateless, behind load balancer
- **Background Workers**: Multiple instances, auto-scaling
- **Database**: Read replicas, connection pooling
- **Cache**: Redis cluster with replication

### 10.2 Performance Optimization
- **Resume Parsing**: Parallel text extraction, caching
- **LLM Calls**: Batch requests where possible, caching results
- **Database**: Indexes on frequently queried fields
- **Frontend**: Code splitting, lazy loading, image optimization

### 10.3 Monitoring & Alerting
- Application Performance Monitoring (Datadog/New Relic)
- Error tracking (Sentry)
- Log aggregation (ELK stack or CloudWatch)
- Alerts for: API response time > 500ms, error rate > 1%, queue depth > 1000

---

## 11. Deployment Architecture

### 11.1 Environments
- **Development**: Local machine, docker-compose
- **Staging**: AWS, mirrors production
- **Production**: Multi-AZ setup, auto-scaling

### 11.2 CI/CD Pipeline
```
Push to GitHub
    ↓
GitHub Actions
    ├─ Lint & Format Check
    ├─ Unit Tests
    ├─ Integration Tests
    └─ Build Docker Image
    ↓
Deploy to Staging (on main branch)
    ↓
Manual Approval
    ↓
Deploy to Production
    ↓
Health Checks & Smoke Tests
```

### 11.3 Infrastructure as Code
- **Terraform**: AWS resources (RDS, S3, ECS, etc.)
- **Docker**: Containerized services
- **Docker Compose**: Local development

---

## 12. Disaster Recovery

### 12.1 Backup Strategy
- **Database**: Daily automated backups, geo-redundant
- **S3 Files**: Versioning enabled, cross-region replication
- **Configuration**: Version controlled in Git

### 12.2 Recovery Procedures
- **RTO**: 4 hours
- **RPO**: 1 hour
- **Failover**: Automated to standby region
- **Testing**: Monthly DR drills

---

## 13. Integration Points

### 13.1 External Services
- **OpenAI/Claude**: LLM API for analysis
- **Stripe**: Payment processing
- **SendGrid**: Email notifications
- **AWS S3**: File storage
- **Pinecone/Weaviate**: Vector embeddings

### 13.2 Future Integrations
- LinkedIn for candidate profiles
- Job board APIs for requirements
- HRIS systems for recruiter workflows
