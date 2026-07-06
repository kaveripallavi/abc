# Development Setup Guide
## AI Resume Analyzer

**Last Updated**: June 2026  
**Status**: Planning Phase

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Environment Setup](#local-environment-setup)
3. [Backend Setup](#backend-setup)
4. [Frontend Setup](#frontend-setup)
5. [Running Locally](#running-locally)
6. [Testing](#testing)
7. [Code Quality](#code-quality)
8. [Git Workflow](#git-workflow)
9. [Deployment Checklist](#deployment-checklist)
10. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required
- **Node.js**: 18.x or higher
  - Install: https://nodejs.org/
  - Verify: `node --version`

- **npm**: 9.x or higher
  - Verify: `npm --version`

- **PostgreSQL**: 14+
  - Install: https://www.postgresql.org/download/
  - Verify: `psql --version`

- **Redis**: 7.x+
  - Install: https://redis.io/download
  - Verify: `redis-cli --version`

- **Docker & Docker Compose**: Latest
  - Install: https://www.docker.com/products/docker-desktop
  - Verify: `docker --version` and `docker-compose --version`

- **Git**: Latest
  - Install: https://git-scm.com/
  - Verify: `git --version`

### Recommended
- **VS Code**: https://code.visualstudio.com/
  - Extensions:
    - Prisma
    - ESLint
    - Prettier
    - Thunder Client or REST Client
    - PostgreSQL

- **Postman**: https://www.postman.com/ (for API testing)

- **pgAdmin**: Web interface for PostgreSQL management

---

## Local Environment Setup

### 1. Clone Repository

```bash
git clone https://github.com/ayushks1ngh/xyz.git
cd xyz
```

### 2. Create Environment Files

**Backend** (`.env`):
```bash
# Server
NODE_ENV=development
PORT=3000
API_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://resumeanalyzer:password@localhost:5432/resumeanalyzer_dev

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_REFRESH_SECRET=your-refresh-token-secret
JWT_EXPIRY=3600
JWT_REFRESH_EXPIRY=604800

# AWS/S3
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
S3_BUCKET=resumeanalyzer-dev

# LLM APIs
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Stripe (for payments)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...

# Email
SENDGRID_API_KEY=SG...

# Logging
LOG_LEVEL=debug

# Feature flags
ENABLE_BATCH_PROCESSING=true
ENABLE_RECRUITER_FEATURES=true
```

**Frontend** (`.env.local`):
```bash
# API Configuration
VITE_API_URL=http://localhost:3000/v1
VITE_API_TIMEOUT=30000

# Feature Flags
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_SENTRY=false

# Optional: Stripe for frontend
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

### 3. Install Dependencies

```bash
# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

---

## Backend Setup

### 1. Initialize Database

```bash
cd backend

# Create database
createdb resumeanalyzer_dev

# Run migrations
npm run prisma:migrate:dev -- --name initial_schema

# Seed database (optional)
npm run prisma:db:seed
```

### 2. Generate Prisma Client

```bash
npm run prisma:generate
```

### 3. Verify Backend

```bash
# Run tests
npm test

# Start development server
npm run dev
```

Should see:
```
Server listening on http://localhost:3000
```

---

## Frontend Setup

### 1. Generate API Types (Optional)

If API schema available:
```bash
cd frontend
npm run generate:types
```

### 2. Start Development Server

```bash
npm run dev
```

Should see:
```
  VITE v4.x.x  ready in xxx ms

  ➜  Local:   http://localhost:5173/
```

---

## Running Locally

### Option A: Manual Process Management

**Terminal 1 - Database**:
```bash
redis-server
```

**Terminal 2 - Backend**:
```bash
cd backend
npm run dev
```

**Terminal 3 - Background Workers** (if using Bull queue):
```bash
cd backend
npm run workers:dev
```

**Terminal 4 - Frontend**:
```bash
cd frontend
npm run dev
```

### Option B: Using Docker Compose (Recommended)

```bash
# From project root
docker-compose up
```

This starts:
- PostgreSQL (port 5432)
- Redis (port 6379)
- Backend (port 3000)
- Frontend (port 5173)

Verify all services:
```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Access Locally

- Frontend: http://localhost:5173
- Backend API: http://localhost:3000/v1
- API Docs: http://localhost:3000/v1/docs (Swagger)
- Redis Commander: http://localhost:6379 (via web UI if installed)
- pgAdmin: http://localhost:5050

---

## Testing

### Backend Tests

```bash
cd backend

# Run all tests
npm test

# Watch mode (re-run on file changes)
npm test -- --watch

# Coverage report
npm test -- --coverage

# Specific test file
npm test -- tests/unit/validators.test.ts

# Integration tests only
npm test -- tests/integration

# Match pattern
npm test -- --testNamePattern="auth"
```

### Frontend Tests

```bash
cd frontend

# Run all tests
npm test

# Watch mode
npm test -- --watch

# Coverage report
npm test -- --coverage

# Specific test file
npm test components/auth/LoginForm.test.tsx
```

### E2E Tests

```bash
cd frontend

# Run Playwright tests (when setup)
npm run test:e2e

# UI mode for debugging
npm run test:e2e -- --ui
```

### Test Structure

```
tests/
├── unit/           # Pure function tests (validators, utils)
├── integration/    # API endpoint tests with database
└── e2e/           # User workflow tests
```

### Writing Tests

**Backend (Jest)**:
```typescript
describe('ResumeService', () => {
  it('should parse resume correctly', async () => {
    const result = await resumeService.parse(testFile);
    expect(result.contact.email).toBe('john@example.com');
  });
});
```

**Frontend (Vitest)**:
```typescript
describe('LoginForm', () => {
  it('should submit form with email and password', () => {
    const { getByRole } = render(<LoginForm />);
    const submitBtn = getByRole('button', { name: /login/i });
    expect(submitBtn).toBeInTheDocument();
  });
});
```

---

## Code Quality

### Linting & Formatting

```bash
cd backend

# Lint check
npm run lint

# Auto-fix lint issues
npm run lint -- --fix

# Format code with Prettier
npm run format

# Check formatting without applying
npm run format:check
```

### Pre-commit Hooks

Automatically installed via husky:

```bash
# Hooks run automatically on git commit
# - Lint staged files
# - Run tests for changed files
# - Format staged files

# Bypass hooks (not recommended)
git commit --no-verify
```

### Type Checking

```bash
cd backend

# TypeScript check
npm run type-check

cd ../frontend
npm run type-check
```

---

## Git Workflow

### Branch Naming

```
feature/feature-name        # New features
bugfix/issue-description    # Bug fixes
refactor/refactor-scope     # Code refactoring
docs/documentation-topic    # Documentation
chore/maintenance-task      # Maintenance, dependencies
```

### Commit Guidelines

```
# Format: <type>(<scope>): <subject>

feat(auth): implement JWT token refresh
fix(resume): handle PDF parsing edge case
docs(api): update endpoint documentation
test(matching): add job matching test suite
refactor(db): optimize resume queries
```

### PR Process

1. Create branch: `git checkout -b feature/my-feature`
2. Make changes and commit regularly
3. Push to remote: `git push -u origin feature/my-feature`
4. Create Pull Request on GitHub
5. Ensure CI/CD passes
6. Request review from team
7. Address review comments
8. Merge to main

### Pushing Code

```bash
# Before push - always pull latest
git pull origin main

# Create/push branch
git push -u origin branch-name

# Push updates
git push origin branch-name

# Force push only for rebasing (use cautiously)
git push --force-with-lease origin branch-name
```

---

## Deployment Checklist

### Pre-deployment

- [ ] All tests passing locally
- [ ] No console errors or warnings
- [ ] Environment variables configured
- [ ] Database migrations tested
- [ ] Code review approved
- [ ] No sensitive data in commits
- [ ] Branch rebased with main

### Staging Deployment

```bash
# Deploy to staging environment
npm run deploy:staging

# Run smoke tests
npm run test:smoke:staging

# Check logs
npm run logs:staging
```

### Production Deployment

```bash
# Create release tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# Deploy (via CI/CD pipeline)
# Deployment automatically triggered by tag push

# Monitor deployment
npm run monitor:production

# Rollback if needed
npm run rollback:production
```

---

## Troubleshooting

### Database Connection Issues

**Problem**: `error: connect ECONNREFUSED 127.0.0.1:5432`

**Solutions**:
```bash
# Check PostgreSQL is running
psql -U postgres -c "SELECT version();"

# Check connection string in .env
# Should be: postgresql://user:password@localhost:5432/dbname

# Test connection manually
psql postgresql://resumeanalyzer:password@localhost:5432/resumeanalyzer_dev

# Reset migrations
npm run prisma:migrate:reset

# Restart PostgreSQL service
# macOS: brew services restart postgresql
# Linux: sudo systemctl restart postgresql
# Windows: net stop PostgreSQL && net start PostgreSQL
```

### Redis Connection Issues

**Problem**: `Error: connect ECONNREFUSED 127.0.0.1:6379`

**Solutions**:
```bash
# Check Redis is running
redis-cli ping
# Should respond: PONG

# Start Redis
redis-server

# Check Redis connection
npm run test:redis
```

### Port Already in Use

**Problem**: `Error: listen EADDRINUSE :::3000`

**Solutions**:
```bash
# Find process using port 3000
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill process
kill -9 <PID>  # macOS/Linux
taskkill /PID <PID> /F  # Windows

# Or use different port
PORT=3001 npm run dev
```

### Module Not Found

**Problem**: `Cannot find module '@/services/resume.service'`

**Solutions**:
```bash
# Reinstall node_modules
rm -rf node_modules package-lock.json
npm install

# Clear TypeScript cache
npm run clean

# Restart dev server
npm run dev
```

### Migration Issues

**Problem**: `Migration failed or stuck`

**Solutions**:
```bash
# Check migration status
npm run prisma:migrate:status

# View specific migration
npm run prisma:migrate:show

# Resolve manually
# 1. Check database state
# 2. Fix schema conflicts
# 3. Re-run: npm run prisma:migrate:deploy

# Nuclear option (development only)
npm run prisma:migrate:reset
```

### Out of Memory

**Problem**: `JavaScript heap out of memory`

**Solutions**:
```bash
# Increase Node memory
NODE_OPTIONS=--max-old-space-size=4096 npm run dev

# Kill background processes
# Close unnecessary browser tabs
# Restart system
```

### API Response Errors

**Problem**: Getting 500 errors from API

**Solutions**:
```bash
# Check backend logs
npm run logs:backend

# Verify environment variables
npm run check:env

# Test specific endpoint
curl http://localhost:3000/v1/health

# Check database connection
npm run test:db
```

### TypeScript Errors After Pull

**Problem**: `TS2345: Argument of type 'X' is not assignable to parameter of type 'Y'`

**Solutions**:
```bash
# Regenerate Prisma types
npm run prisma:generate

# Clear TypeScript cache
npm run clean

# Type check
npm run type-check

# Restart IDE/editor
```

---

## Common Development Tasks

### Adding a New Endpoint

1. Define types in `backend/src/types/`
2. Add route in `backend/src/routes/`
3. Implement service in `backend/src/services/`
4. Write tests in `backend/tests/`
5. Document in `docs/API.md`

### Modifying Database Schema

1. Update Prisma schema: `backend/prisma/schema.prisma`
2. Create migration: `npm run prisma:migrate:dev -- --name meaningful_name`
3. Review generated SQL
4. Test locally
5. Add migration notes to docs

### Adding a New Component

1. Create component file: `frontend/src/components/MyComponent.tsx`
2. Export from index: `frontend/src/components/index.ts`
3. Add TypeScript types
4. Write unit tests
5. Add Storybook story (optional)

### Performance Profiling

**Backend**:
```bash
# Use clinic.js
npm install -g clinic

# Profile CPU
clinic doctor -- node dist/server.js

# View report
# Results show in browser automatically
```

**Frontend**:
```bash
# React DevTools browser extension
# Run build preview
npm run build
npm run preview

# Check bundle size
npm run build:analyze
```

---

## Resources & Documentation

### Internal
- [Product Requirements](./docs/PRD.md)
- [Architecture Design](./docs/ARCHITECTURE.md)
- [API Reference](./docs/API.md)
- [Database Schema](./docs/DATABASE.md)
- [Technical Decisions](./docs/DECISIONS.md)

### External
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [React Best Practices](https://react.dev/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Team Communication

- **Slack**: #development channel
- **GitHub Issues**: For bugs and features
- **Architecture Reviews**: Every 2 weeks
- **Code Review Guidelines**: Check CONTRIBUTING.md (create when needed)

---

## Getting Help

1. Check this guide's troubleshooting section
2. Search GitHub issues for similar problems
3. Ask in team Slack channel
4. Create new GitHub issue with details

**When reporting issues, include**:
- Steps to reproduce
- Error message/logs
- Environment (OS, Node version, etc.)
- Screenshots (if UI related)

