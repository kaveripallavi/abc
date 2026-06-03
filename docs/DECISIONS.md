# Technical Decisions Document
## AI Resume Analyzer

**Version**: 1.0  
**Status**: Planning  
**Last Updated**: June 2026

---

## Overview

This document captures key architectural and technology decisions, the rationale behind them, and alternatives considered.

---

## 1. Tech Stack Decisions

### 1.1 Backend: Node.js + Express

**Decision**: Use Node.js with Express for backend development.

**Rationale**:
- Strong ecosystem for file handling and text processing
- Excellent async/await support for parallel processing
- Fast development velocity with TypeScript
- JavaScript/TypeScript across stack reduces context switching
- Large community and numerous libraries (pdf-parse, docx, NLP tools)
- Good performance for I/O-bound operations (parsing, API calls)

**Alternatives Considered**:
- **Python + FastAPI**: Better for ML/NLP, but heavier for file operations, less suitable for real-time requirements
- **Go**: Excellent performance but steeper learning curve for team, smaller ecosystem for resume parsing
- **Java/Spring**: Overkill complexity for MVP, slower development

**Trade-offs**:
- Node.js: Fast dev, good performance, smaller for CPU-intensive tasks (handled by external LLM APIs)

---

### 1.2 Frontend: React + Vite + TypeScript

**Decision**: React with Vite build tool and TypeScript for frontend.

**Rationale**:
- React dominates job seeker market (familiarity, talent pool)
- Vite provides exceptional development experience (< 100ms HMR)
- TypeScript catches errors early, improves maintainability
- shadcn/ui provides professional component library quickly
- Ecosystem fully supports complex state management and async operations

**Alternatives Considered**:
- **Vue.js**: Easier to learn but smaller ecosystem, less talent pool
- **Next.js**: Adds complexity for MVP, not needed (SPA is sufficient)
- **Svelte**: Excellent but smaller community, less hiring pool

**Trade-offs**:
- React + Vite: Great dev experience, requires component discipline, bundle size management needed

---

### 1.3 Database: PostgreSQL

**Decision**: PostgreSQL as primary relational database.

**Rationale**:
- ACID compliance critical for payment/subscription data
- Excellent JSON support for storing parsed resume structure
- Full-text search for resume content searching
- Proven reliability and scalability
- Managed service available on all major cloud providers
- Strong ORM support (Prisma)

**Alternatives Considered**:
- **MongoDB**: More flexible schema but worse for relational data (users, subscriptions)
- **MySQL**: Good but lacks advanced features needed (JSON, FTS)
- **DynamoDB**: Good for scale but overkill for MVP, more complex queries

**Trade-offs**:
- PostgreSQL: Schema-first approach requires migration management but provides safety

---

### 1.4 ORM: Prisma

**Decision**: Use Prisma as ORM for database abstraction.

**Rationale**:
- Modern, TypeScript-first design
- Excellent developer experience with auto-completion
- Built-in migrations system
- Strong relationship querying
- Type-safe queries prevent SQL injection
- Seamless integration with TypeScript types

**Alternatives Considered**:
- **TypeORM**: More powerful but heavier, steeper learning curve
- **Sequelize**: Older, less modern DX
- **Raw SQL**: Too error-prone, harder to maintain

**Trade-offs**:
- Prisma: Less flexible for complex queries but safer and faster to develop

---

### 1.5 Job Queue: Bull with Redis

**Decision**: Bull queue for background job processing, backed by Redis.

**Rationale**:
- Resume parsing, analysis, and batch processing are background jobs
- Bull provides reliable job queuing with retry logic
- Redis is fast, widely supported, easily scalable
- Clear separation of concerns: API fast, heavy lifting in workers
- Essential for handling 100+ resumes without blocking API

**Alternatives Considered**:
- **AWS SQS**: More managed but vendor lock-in, more latency
- **RabbitMQ**: More complex setup, overkill for MVP
- **Temporal/Temporal Cloud**: More sophisticated but unnecessary complexity

**Trade-offs**:
- Bull: Adds operational complexity but necessary for scalability

---

### 1.6 File Storage: AWS S3 (or Cloud Provider Equivalent)

**Decision**: Use cloud object storage (S3) for resume file storage.

**Rationale**:
- Scalable, reliable, cost-effective
- Integrates with IAM for security
- Versioning support for audit trails
- CDN integration available
- Easy backup and disaster recovery
- Offloads file handling from application server

**Alternatives Considered**:
- **Local filesystem**: Doesn't scale, difficult to backup, not suitable for distributed setup
- **Database blob storage**: More expensive, slower, complicates backups

**Trade-offs**:
- S3: Adds cloud vendor dependency but essential for production

---

### 1.7 LLM Provider: OpenAI GPT-4 / Claude 3

**Decision**: Use OpenAI or Anthropic APIs for AI analysis.

**Rationale**:
- GPT-4 and Claude 3 show superior reasoning for text analysis
- Managed service (no training/hosting required)
- APIs available with reliable uptime
- Cost-effective at scale
- Both support function calling for structured extraction

**Alternatives Considered**:
- **Open source models (Llama 2, Mistral)**: Requires hosting, more ops work, comparable quality
- **Google Gemini**: Less established, fewer integrations
- **Local models**: Complex to maintain, lower quality for specialized tasks

**Trade-offs**:
- Vendor dependency on API availability and pricing but faster time-to-market

---

### 1.8 Vector Database: Pinecone or Weaviate

**Decision**: Use managed vector DB for embeddings and semantic search.

**Rationale**:
- Essential for semantic job matching (embedding similarity)
- Managed service reduces ops complexity
- Pinecone has excellent performance for resume-size embeddings
- Weaviate is open-source alternative

**Alternatives Considered**:
- **Redis with similarity search module**: Lighter but less powerful
- **pgvector (PostgreSQL extension)**: Works but adds complexity
- **Elasticsearch**: Overkill, more for full-text search

**Trade-offs**:
- Managed vector DB: Adds cost but simplifies architecture

---

## 2. Architectural Decisions

### 2.1 Microservices vs Monolith

**Decision**: Start with monolithic backend, ready to split services later.

**Rationale**:
- Faster MVP development
- Easier debugging and deployment
- Sufficient for initial 100K users
- Can be decomposed to microservices if needed (Phase 3+)
- Simpler operational requirements

**Alternatives Considered**:
- **Full microservices**: Adds complexity, overkill for MVP
- **Serverless functions**: Cold start issues for long-running parsing jobs

**Trade-offs**:
- Monolith: Eventually needs splitting but faster to ship

---

### 2.2 API Design: REST with JSON

**Decision**: RESTful API with JSON request/response.

**Rationale**:
- Standard, widely understood
- Simple to test and document
- Sufficient for current requirements
- GraphQL not necessary without complex querying needs

**Alternatives Considered**:
- **GraphQL**: Adds complexity, query optimization needed
- **gRPC**: Overkill for web app

**Trade-offs**:
- REST: Over/under-fetching possible but acceptable trade-off

---

### 2.3 Real-time Features: Not in MVP

**Decision**: Queue-based async processing, no WebSockets initially.

**Rationale**:
- Analysis and matching don't require real-time updates
- Users can poll status or receive emails
- Adds complexity with WebSocket infrastructure
- Sufficient to ship MVP

**Alternatives Considered**:
- **WebSockets**: Real-time progress updates but added complexity
- **Server-Sent Events (SSE)**: Middle ground, could add later

**Trade-offs**:
- Polling: Simpler architecture but slightly worse UX (Phase 2 upgrade)

---

### 2.4 Caching Strategy: Redis + CDN

**Decision**: Multi-layer caching: Redis for application cache, CDN for static assets.

**Rationale**:
- Resume parsing results are expensive to recompute
- Analysis scores don't change frequently
- Cache invalidation on update
- CDN for images, CSS, JavaScript

**Trade-offs**:
- Cache invalidation complexity but huge performance gains

---

### 2.5 Search: Full-text PostgreSQL + Semantic Vector DB

**Decision**: PostgreSQL full-text search for resume content, vector embeddings for semantic matching.

**Rationale**:
- Full-text search built into PostgreSQL (no external dependency)
- Vector embeddings for job matching (semantic meaning)
- Two approaches for different search needs

**Trade-offs**:
- Two databases: Adds complexity but each excels at its purpose

---

## 3. Security Decisions

### 3.1 Authentication: JWT with Refresh Token Rotation

**Decision**: JWT-based authentication with short-lived access tokens and refresh tokens.

**Rationale**:
- Stateless, scalable across instances
- Refresh token rotation improves security
- Standard approach, well-understood
- Avoids session store complexity

**Alternatives Considered**:
- **Session-based**: Requires session store, doesn't scale as well
- **OAuth 2.0**: Overkill for MVP, can add as alternative login later

**Trade-offs**:
- JWT: Tokens are not revocable immediately but refresh token rotation mitigates

---

### 3.2 Password Hashing: bcrypt

**Decision**: Use bcrypt for password hashing.

**Rationale**:
- Proven, slow hashing algorithm designed for passwords
- Resistant to GPU brute-force attacks
- Industry standard

**Trade-offs**:
- Slight performance impact (intentional) but negligible for login

---

### 3.3 Data Encryption: AES-256 at Rest, TLS 1.3 in Transit

**Decision**: Encrypt sensitive data at rest in database, enforce TLS 1.3 for all transport.

**Rationale**:
- PII (resumes, contact info) must be encrypted
- Compliance with security standards
- TLS 1.3 is modern, performant

**Trade-offs**:
- Minimal performance impact, significant security benefit

---

### 3.4 RBAC: Role-Based Access Control

**Decision**: Implement RBAC with User, Recruiter, Admin roles.

**Rationale**:
- Clear permission boundaries
- Subscription tiers tied to roles/features
- Scalable for future permission complexity

**Trade-offs**:
- Adds permission check overhead but minimal performance impact

---

## 4. Development Workflow Decisions

### 4.1 Version Control: Git with GitHub

**Decision**: Git with GitHub for version control and CI/CD.

**Rationale**:
- Industry standard
- Excellent integration with CI/CD tools
- Strong collaboration features
- Large user base for team familiarity

---

### 4.2 Code Organization: Monorepo Structure

**Decision**: Single repository with backend/ and frontend/ directories.

**Rationale**:
- Easier dependency management
- Single CI/CD pipeline
- Shared type definitions possible
- Simple for MVP size

**Alternatives Considered**:
- **Separate repos**: More complex setup, CI/CD complexity
- **Monorepo tools (Turborepo)**: Overkill for MVP

---

### 4.3 Testing: Jest + Vitest

**Decision**: Jest for backend, Vitest for frontend unit tests.

**Rationale**:
- Jest: Batteries-included, mature, great for Node.js
- Vitest: Lightning-fast, modern, excellent Vite integration
- Both have excellent TypeScript support

**Trade-offs**:
- Two test runners but each is optimized for environment

---

### 4.4 Code Quality: ESLint + Prettier

**Decision**: ESLint for linting, Prettier for formatting.

**Rationale**:
- Prevents bugs through static analysis
- Consistent code style
- Auto-fixable issues
- Git pre-commit hooks for enforcement

---

### 4.5 CI/CD: GitHub Actions

**Decision**: GitHub Actions for CI/CD pipeline.

**Rationale**:
- Native GitHub integration
- No external vendor for MVP
- Sufficient for current scale
- Can migrate to other tools later if needed

---

## 5. Data & Analytics Decisions

### 5.1 Event Tracking: Minimal Initially

**Decision**: Track core events (upload, analysis, match) without external analytics initially.

**Rationale**:
- Keep MVP simple
- Implement logging in application
- Add Mixpanel/Segment in Phase 2

**Trade-offs**:
- Limited initial analytics but sufficient to measure success metrics

---

### 5.2 Monitoring: Application Monitoring Tool (Phase 2)

**Decision**: Defer advanced monitoring to Phase 2, use basic logging initially.

**Rationale**:
- MVP doesn't have complex operational needs
- Basic logging sufficient
- Add Datadog/New Relic when scaling

---

## 6. UI/UX Decisions

### 6.1 Component Library: shadcn/ui

**Decision**: Use shadcn/ui for UI components.

**Rationale**:
- Built on Radix UI (accessible primitives)
- Tailwind CSS integration
- Customizable, not opinionated
- Copy-paste approach (own components)
- Professional look quickly

**Alternatives Considered**:
- **Material-UI**: Larger bundle, more opinionated
- **Chakra UI**: Good but shadcn/ui integrates better with Tailwind

---

### 6.2 Styling: Tailwind CSS

**Decision**: Tailwind CSS for styling.

**Rationale**:
- Utility-first approach speeds development
- Small file size with PurgeCSS
- Easy to customize
- Large ecosystem of Tailwind components

---

### 6.3 Icons: Lucide React

**Decision**: Use Lucide React for icons.

**Rationale**:
- Modern, clean icon set
- React component format
- Easily customizable (color, size)
- Consistent with shadcn/ui

---

## 7. Deployment Decisions

### 7.1 Container Orchestration: Docker (Single Server MVP)

**Decision**: Use Docker for local dev and single container on server initially.

**Rationale**:
- Consistency across dev/prod
- Easy to scale later with Kubernetes
- Simpler setup for MVP

**Future**: Migrate to ECS/EKS when scaling

---

### 7.2 Infrastructure Provider: AWS (Primary)

**Decision**: AWS as primary cloud provider.

**Rationale**:
- Comprehensive service portfolio
- Mature PostgreSQL (RDS) and Redis (ElastiCache)
- S3 for storage
- CDN (CloudFront) available
- Can scale easily

**Alternatives**: Azure or GCP acceptable with similar capabilities

---

### 7.3 Database Backups: Automated Daily

**Decision**: Automated daily backups with 30-day retention.

**Rationale**:
- Recovery Point Objective (RPO) of 24 hours acceptable for MVP
- Geo-redundant backups for disaster recovery
- Cost-effective with cloud provider managed backups

---

## 8. Cost Optimization Decisions

### 8.1 LLM API Costs

**Decision**: Cache analysis results, batch requests where possible.

**Rationale**:
- Same resume analyzed multiple times → cache for 60 days
- LLM costs could easily become 50%+ of revenue if not managed
- Implement usage quotas per tier

**Impact**: Reduces LLM costs by estimated 60-70%

---

### 8.2 Storage Costs

**Decision**: Compress PDFs, delete old temporary files.

**Rationale**:
- 1000 resumes at 500KB each = 500GB storage
- Implement S3 lifecycle policies
- Delete parsed text after 30 days if user doesn't interact

**Impact**: ~30% reduction in storage costs

---

### 8.3 Database Costs

**Decision**: Use read replicas only when hitting read limits (Phase 2+).

**Rationale**:
- Single primary sufficient for MVP
- Read replicas expensive, add when needed
- Application design allows easy migration

---

## 9. Future Decision Points

### 9.1 Scaling to Microservices (Phase 2-3)

**Timeline**: When MVP reaches 10K+ users or system becomes difficult to maintain

**Candidates for decomposition**:
1. Parse Service (heavy CPU/I/O)
2. Analysis Service (LLM calls)
3. Matching Service (vector search)
4. Recruiter Batch Service (high concurrency)

---

### 9.2 Adding Real-time Features (Phase 2-3)

**Timeline**: Based on user feedback and mobile app needs

**Options**:
- WebSockets for live analysis progress
- Server-Sent Events for lighter updates
- GraphQL subscriptions

---

### 9.3 Mobile App (Phase 3+)

**Decision**: React Native for cross-platform
**Timeline**: After web MVP proven successful
**Rationale**: Share business logic with web, faster development

---

### 9.4 Machine Learning Model

**Timeline**: Phase 3+ based on data collection

**Opportunities**:
- Resume quality prediction model
- Better skill extraction (fine-tuned model)
- Candidate ranking optimization
- Job matching model

---

## 10. Decision Review Process

### Quarterly Review
- Evaluate tech stack choices against alternatives
- Assess cost structure
- Review performance bottlenecks
- Plan Phase N+1 architecture

### Annual Review
- Major architectural decisions revisit
- Technology upgrade assessment
- Scaling strategy review

---

## Appendix: Decision Template

For future technical decisions, use this template:

```
## [Decision Number]. [Decision Title]

**Decision**: [What was decided]

**Rationale**:
- [Reason 1]
- [Reason 2]
- [Reason 3]

**Alternatives Considered**:
- [Alternative 1]: [Why not chosen]
- [Alternative 2]: [Why not chosen]

**Trade-offs**:
- [Trade-off 1]
- [Trade-off 2]

**Review Date**: [Date to reconsider]
```

---

## Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-06-03 | Initial planning phase |
