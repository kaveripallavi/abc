# Quick Start Reference
## AI Resume Analyzer - Planning Phase Complete

This is a quick reference for the AI Resume Analyzer project. Full documentation is available in the `/docs` directory.

---

## 📚 Documentation Map (Start Here!)

| Role | Start With | Then Read |
|------|-----------|-----------|
| **Product Manager** | [PRD.md](./docs/PRD.md) | [TASKS.md](./docs/TASKS.md), [DECISIONS.md](./docs/DECISIONS.md) |
| **Architect** | [ARCHITECTURE.md](./docs/ARCHITECTURE.md) | [DATABASE.md](./docs/DATABASE.md), [DECISIONS.md](./docs/DECISIONS.md) |
| **Backend Engineer** | [DEVELOPMENT.md](./DEVELOPMENT.md) | [API.md](./docs/API.md), [DATABASE.md](./docs/DATABASE.md) |
| **Frontend Engineer** | [DEVELOPMENT.md](./DEVELOPMENT.md) | [API.md](./docs/API.md), [PRD.md](./docs/PRD.md) |
| **DevOps** | [ARCHITECTURE.md](./docs/ARCHITECTURE.md) | [DATABASE.md](./docs/DATABASE.md), [DEVELOPMENT.md](./DEVELOPMENT.md) |
| **New to Project** | [docs/README.md](./docs/README.md) | Your role-specific path above |

---

## 🚀 Project Overview

**Name**: AI Resume Analyzer  
**Phase**: MVP Planning Complete  
**Timeline**: 8-10 weeks to MVP  
**Team Size**: 8 people (3 backend, 2 frontend, 1 DevOps, 2 Product/Design)

### What We're Building
An intelligent SaaS platform that helps job seekers optimize resumes and enables recruiters to screen candidates using AI-powered analysis, scoring, and matching.

### Key Features (MVP)
- ✅ Resume upload & parsing (PDF, DOCX, TXT)
- ✅ Quality analysis with scoring
- ✅ Job description matching
- ✅ Skill gap analysis
- ✅ Personalized recommendations
- ✅ Recruiter batch processing (Phase 2)

---

## 🏗️ Architecture at a Glance

```
Frontend (React + Vite)   →   Backend (Node + Express)   →   Database (PostgreSQL)
                              ↓
                         Workers (Bull Queue)
                              ↓
                         External APIs (LLM, S3, Stripe)
```

**Tech Stack**:
- Backend: Node.js + Express + TypeScript
- Frontend: React 18 + Vite + Tailwind CSS
- Database: PostgreSQL 14+
- Cache: Redis
- Queue: Bull
- LLM: OpenAI/Claude
- Storage: AWS S3
- Deployment: Docker + GitHub Actions

---

## 📅 Sprint Breakdown

| Sprint | Duration | Focus |
|--------|----------|-------|
| 1 | Week 1-2 | Backend setup, database, Docker, CI/CD |
| 2 | Week 2-3 | Authentication & user management |
| 3 | Week 3-5 | Resume upload & parsing |
| 4 | Week 5-7 | Quality analysis engine |
| 5 | Week 7-9 | Job matching core |
| 6 | Week 9-10 | Polish, testing, launch prep |

---

## 🔧 Getting Started

### Prerequisites
```bash
Node.js 18+, npm 9+, PostgreSQL 14+, Redis 7+, Docker
```

### Local Setup
```bash
# Clone and install
git clone https://github.com/ayushks1ngh/xyz.git
cd xyz

# Copy environment
cp .env.example .env

# Start services (Docker)
docker-compose up

# Or manual setup (see DEVELOPMENT.md)
```

### Verify Setup
```bash
# Backend
curl http://localhost:3000/health

# Frontend  
open http://localhost:5173

# Docs
open http://localhost:3000/v1/docs
```

---

## 📖 Key Documents

### Planning & Strategy
- **PRD.md** - What we're building (users, features, pricing)
- **TASKS.md** - How to build it (sprints, timeline, tasks)
- **DECISIONS.md** - Why we chose it (tech decisions, trade-offs)

### Technical Design
- **ARCHITECTURE.md** - System design, tech stack, folder structure
- **DATABASE.md** - Schema, tables, indexes, migrations
- **API.md** - Endpoints, formats, error handling

### Development
- **DEVELOPMENT.md** - Setup, testing, deployment
- **CONTRIBUTING.md** - Code style, PR process, guidelines

### Navigation
- **docs/README.md** - Complete documentation index
- **QUICK_START.md** - This file!

---

## 🎯 Key Endpoints (MVP)

### Authentication
```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
```

### Resume Management
```
POST   /resumes/upload          # Upload resume
GET    /resumes                 # List resumes
GET    /resumes/:id             # Get resume details
DELETE /resumes/:id             # Delete resume
```

### Analysis
```
POST   /analysis/quality        # Analyze quality
POST   /analysis/ats-score      # ATS score
GET    /analysis/:id            # Get results
```

### Job Matching
```
POST   /matching/score          # Score vs job
GET    /matching/:id            # Get match results
POST   /skills/gap-analysis     # Skill gaps
GET    /skills/gap-analysis/:id # Get gap results
```

See [API.md](./docs/API.md) for complete reference.

---

## 🗄️ Database Tables

| Table | Purpose | Key Fields |
|-------|---------|-----------|
| users | User accounts | id, email, user_type, subscription_tier |
| resumes | Uploaded files | id, user_id, file_url, parsing_status |
| analyses | Analysis results | id, resume_id, quality_score, feedback |
| jobs | Job descriptions | id, title, requirements, user_id |
| matches | Job-resume matches | id, resume_id, job_id, match_score |
| skill_gaps | Skill gap analysis | id, resume_id, target_role, missing_skills |
| analysis_batches | Recruiter batch jobs | id, user_id, status, total_resumes |
| subscriptions | Payment info | id, user_id, tier, stripe_id |

See [DATABASE.md](./docs/DATABASE.md) for complete schema.

---

## ⚙️ Environment Variables

**Critical for Development**:
```bash
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@localhost:5432/db
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key
OPENAI_API_KEY=sk-...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

See [.env.example](./.env.example) for full list.

---

## 📊 Success Criteria

### MVP Success (Week 10)
- ✅ Users can upload & analyze resumes
- ✅ Resume quality scores (0-100)
- ✅ Job matching with gap analysis
- ✅ 70%+ test coverage
- ✅ Can handle 100+ concurrent users

### Phase 2 (Month 4)
- ✅ Batch resume processing
- ✅ Recruiter features
- ✅ Candidate comparison

### Phase 3 (Month 5)
- ✅ Skill gap learning paths
- ✅ Public API
- ✅ Ready for enterprise

---

## 🔐 Security Essentials

- ✅ JWT authentication (access + refresh tokens)
- ✅ Password hashing (bcrypt)
- ✅ HTTPS/TLS 1.3
- ✅ PII encryption at rest
- ✅ Rate limiting (100 req/min)
- ✅ Input validation (Zod)
- ✅ SQL injection prevention (Prisma)

See [DECISIONS.md](./docs/DECISIONS.md) Section 3 for details.

---

## 🧪 Testing Quick Commands

```bash
# Backend
npm test                          # Run all tests
npm test -- --coverage           # Coverage report
npm test -- --watch             # Watch mode
npm run lint                      # Lint check
npm run type-check               # TypeScript check

# Frontend
npm test                         # Run tests
npm test -- --coverage          # Coverage report
npm run build                    # Production build
npm run build:analyze           # Bundle size
```

---

## 📤 Deployment Checklist

Before pushing to production:
- [ ] All tests pass locally
- [ ] No console errors
- [ ] Environment variables set
- [ ] Database migrations run
- [ ] Code reviewed
- [ ] Performance tested
- [ ] Accessibility checked
- [ ] Security review done

---

## 🆘 Common Issues

### Can't connect to database?
```bash
# Check PostgreSQL running
psql -c "SELECT version();"

# Check .env DATABASE_URL
# Should be: postgresql://user:pass@localhost:5432/db

# Reset migrations (dev only)
npm run prisma:migrate:reset
```

### Port already in use?
```bash
# Find process
lsof -i :3000      # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill process or use different port
PORT=3001 npm run dev
```

### Tests failing?
```bash
# Reinstall dependencies
rm -rf node_modules
npm install

# Clear cache
npm run clean

# Run tests with verbose output
npm test -- --verbose
```

See [DEVELOPMENT.md](./DEVELOPMENT.md#troubleshooting) for more.

---

## 🤝 Contributing

1. Create branch: `git checkout -b feature/my-feature`
2. Follow code style: `npm run lint`
3. Write tests: `npm test`
4. Commit: `git commit -m "feat: description"`
5. Push: `git push -u origin feature/my-feature`
6. Create PR on GitHub

See [CONTRIBUTING.md](./CONTRIBUTING.md) for full guidelines.

---

## 📞 Getting Help

- **Setup Issues**: [DEVELOPMENT.md](./DEVELOPMENT.md#troubleshooting)
- **Documentation Index**: [docs/README.md](./docs/README.md)
- **API Reference**: [docs/API.md](./docs/API.md)
- **Architecture Questions**: [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)

---

## 📋 Documentation Checklist

✅ **Complete Planning**
- [x] PRD - Product requirements
- [x] ARCHITECTURE - System design
- [x] DATABASE - Data model
- [x] API - Endpoint specification
- [x] TASKS - Implementation roadmap
- [x] DECISIONS - Tech choices

✅ **Complete Development Resources**
- [x] DEVELOPMENT - Setup & deployment
- [x] CONTRIBUTING - Code standards
- [x] QUICK_START - This reference
- [x] .env.example - Configuration template

✅ **Complete Organization**
- [x] docs/README.md - Navigation guide
- [x] All documents cross-referenced
- [x] 4 professional git commits
- [x] Pushed to GitHub

---

## 🎓 Learning Path

**New to the project?** Follow this path:

1. Read **QUICK_START.md** (you are here!)
2. Skim **PRD.md** to understand what we're building
3. Review **ARCHITECTURE.md** section 1 for system overview
4. Follow **DEVELOPMENT.md** to set up locally
5. Read your role-specific section in **docs/README.md**
6. Deep-dive into specific documents as needed

---

## 🚀 Ready to Code?

Everything is planned and documented. You can now:

1. **Set up**: Follow [DEVELOPMENT.md](./DEVELOPMENT.md)
2. **Code**: Follow [CONTRIBUTING.md](./CONTRIBUTING.md)
3. **Reference**: Use [API.md](./docs/API.md) and [DATABASE.md](./docs/DATABASE.md)
4. **Track**: Follow [TASKS.md](./docs/TASKS.md) sprints

**Let's build! 🎉**

---

**Created**: June 3, 2026  
**Status**: Ready for MVP Development  
**Next**: Implementation Sprint 1 begins  
**Questions?** Check [docs/README.md](./docs/README.md)
