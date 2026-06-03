# Contributing Guidelines
## AI Resume Analyzer

**Version**: 1.0  
**Status**: Active

---

## Code of Conduct

- Be respectful and inclusive
- Focus on the code, not the person
- Help others learn and grow
- Report concerns to project maintainers

---

## Before You Start

1. Ensure you have the latest version: `git pull origin main`
2. Create a new branch: `git checkout -b feature/my-feature`
3. Follow the [Development Setup Guide](../DEVELOPMENT.md)
4. Install pre-commit hooks: `npm run prepare`

---

## Code Style & Standards

### TypeScript

```typescript
// Use explicit types
const userId: string = "user-123";

// Use const by default
const config = { timeout: 5000 };

// Descriptive names
const calculateResumeQualityScore = (resume: Resume): number => {
  // ...
};

// Use interfaces for contracts
interface AnalysisResult {
  qualityScore: number;
  feedback: string[];
  recommendations: Recommendation[];
}
```

### ESLint & Prettier

```bash
# Format code before commit
npm run format

# Check for issues
npm run lint

# Auto-fix issues
npm run lint -- --fix
```

### Comments & Documentation

```typescript
/**
 * Analyzes resume quality and generates feedback
 * @param resume - Parsed resume content
 * @param userTier - Subscription tier affecting analysis depth
 * @returns Analysis result with scores and recommendations
 * @throws Error if resume parsing fails
 * @example
 * const result = await analyzeResume(resume, 'pro');
 */
export async function analyzeResume(
  resume: Resume,
  userTier: SubscriptionTier
): Promise<AnalysisResult> {
  // Implementation
}
```

---

## Testing Requirements

### Unit Tests

- Write tests for new functions
- Aim for 80%+ coverage on new code
- Use descriptive test names

```typescript
describe('ResumeService', () => {
  describe('parseResume', () => {
    it('should extract contact information correctly', () => {
      // Arrange
      const testFile = loadFixture('resume.pdf');
      
      // Act
      const result = parseResume(testFile);
      
      // Assert
      expect(result.contact.email).toBe('john@example.com');
    });

    it('should throw error for invalid file type', () => {
      const invalidFile = { content: 'test', type: 'exe' };
      expect(() => parseResume(invalidFile)).toThrow();
    });
  });
});
```

### Integration Tests

- Test API endpoints with database
- Clean up test data after each test
- Use transactions for test isolation

```typescript
describe('POST /resumes/upload', () => {
  let testUser: User;

  beforeEach(async () => {
    testUser = await createTestUser();
  });

  afterEach(async () => {
    await cleanupTestData();
  });

  it('should upload and parse resume', async () => {
    const response = await request(app)
      .post('/resumes/upload')
      .auth(testUser.token)
      .attach('file', './test-resume.pdf');

    expect(response.status).toBe(202);
    expect(response.body.data.status).toBe('pending');
  });
});
```

### Running Tests

```bash
# All tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage

# Specific file
npm test -- resume.service.test.ts

# Specific test
npm test -- --testNamePattern="should parse"
```

---

## Git Workflow

### Branch Naming

```
feature/user-authentication          # New feature
bugfix/resume-parsing-edge-case      # Bug fix
refactor/api-response-format         # Code cleanup
docs/architecture-diagrams           # Documentation
chore/update-dependencies            # Maintenance
```

### Commit Messages

Follow this format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Scope**: Component or module affected (optional but recommended)

**Subject**: 
- Imperative mood ("add" not "adds")
- Don't capitalize first letter
- No period at end
- Max 50 characters

**Body** (optional):
- Explain what and why, not how
- Wrap at 72 characters
- Use bullet points for multiple items

**Footer** (optional):
- Reference issues: `Fixes #123`
- Breaking changes: `BREAKING CHANGE: description`

### Examples

```
feat(auth): implement JWT token refresh

- Add refresh token rotation
- Implement automatic token renewal
- Add secure token storage

Fixes #123
```

```
fix(resume): handle PDF with images correctly

Previously, PDFs with embedded images would fail parsing.
Updated text extraction to skip image data and focus on text nodes.

Fixes #456
```

### Pushing Code

```bash
# Create feature branch
git checkout -b feature/my-feature

# Commit regularly (not huge commits)
git add .
git commit -m "feat(feature): implementation step 1"

# Push when ready for review
git push -u origin feature/my-feature

# Create Pull Request on GitHub
# -> fill in template
# -> link related issues
# -> request reviewers

# Update branch if needed
git fetch origin
git rebase origin/main
git push --force-with-lease
```

---

## Pull Request Process

### Before Submitting

- [ ] Code follows style guide
- [ ] Tests added for new features
- [ ] All tests pass: `npm test`
- [ ] No lint errors: `npm run lint`
- [ ] Types check: `npm run type-check`
- [ ] Documentation updated
- [ ] No sensitive data in commits
- [ ] Commits are squashed/clean

### PR Description Template

```markdown
## Description
Brief summary of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation

## Related Issues
Fixes #123

## Changes Made
- Change 1
- Change 2

## Testing
Describe how to test these changes

## Screenshots/Video (if UI change)
[Add screenshots or video]

## Performance Impact
None / [Describe impact]

## Checklist
- [ ] Tests pass locally
- [ ] Code follows style guide
- [ ] Documentation updated
```

### Code Review Process

1. **Automated Checks**
   - Lint: Must pass
   - Tests: Must pass (80%+ coverage)
   - Type check: Must pass
   - Build: Must succeed

2. **Peer Review** (at least 1 required)
   - Code quality
   - Design consistency
   - Test adequacy
   - Documentation completeness

3. **Approval & Merge**
   - At least 1 approval required
   - All feedback addressed
   - All checks passing
   - Can be merged by maintainers

### Common Review Comments

**Suggest**: "Consider using destructuring here"  
**Request Change**: "This needs to handle null values"  
**Question**: "Why is this condition needed?"

---

## Adding Features

### Feature Checklist

- [ ] Update PRD if scope changes
- [ ] Update ARCHITECTURE.md if needed
- [ ] Update DATABASE.md if schema changes
- [ ] Create API endpoints (see API.md)
- [ ] Implement backend service
- [ ] Write backend tests (unit + integration)
- [ ] Create frontend components
- [ ] Write frontend tests
- [ ] Update API documentation
- [ ] Create migration if needed
- [ ] Test end-to-end workflow
- [ ] Performance tested
- [ ] Accessibility checked
- [ ] Security review if needed

### Example: Add Skill Recommendation Feature

1. **Planning**
   - Review [PRD.md](./docs/PRD.md) for requirements
   - Check [ARCHITECTURE.md](./docs/ARCHITECTURE.md) for design
   - Update [TASKS.md](./docs/TASKS.md) with implementation steps

2. **Database**
   - Update Prisma schema: `backend/prisma/schema.prisma`
   - Create migration: `npm run prisma:migrate:dev -- --name add_skill_recommendations`
   - Update [DATABASE.md](./docs/DATABASE.md)

3. **Backend**
   - Create service: `backend/src/services/skillRecommendation.service.ts`
   - Create routes: `backend/src/routes/skills.ts`
   - Write tests: `backend/tests/integration/skills.test.ts`
   - Update API documentation: [API.md](./docs/API.md)

4. **Frontend**
   - Create components: `frontend/src/components/skills/`
   - Create pages: `frontend/src/pages/SkillRecommendations.tsx`
   - Write tests: `frontend/tests/components/skills/`
   - Update types: `frontend/src/types/`

5. **Testing**
   - Run all tests: `npm test`
   - Test locally end-to-end
   - Performance check

6. **Documentation**
   - Update API.md with new endpoints
   - Add developer notes if complex
   - Update this CONTRIBUTING.md if needed

---

## Reporting Issues

### Bug Report Template

```markdown
**Description**
Clear description of the bug

**Steps to Reproduce**
1. Step 1
2. Step 2
3. Bug occurs

**Expected Behavior**
What should happen

**Actual Behavior**
What actually happens

**Environment**
- OS: [Windows/macOS/Linux]
- Node version: [output of `node --version`]
- Browser: [if frontend issue]

**Logs/Screenshots**
[Attach error messages or screenshots]

**Additional Context**
Any other relevant information
```

### Feature Request Template

```markdown
**Description**
Clear description of the feature

**Use Case**
Why is this feature needed?

**Acceptance Criteria**
- Criterion 1
- Criterion 2

**Proposed Solution**
How should this be implemented? (optional)

**Alternatives Considered**
Other ways to solve this (optional)
```

---

## Documentation

### Update API Documentation

When adding/modifying endpoints:

1. Update [docs/API.md](./docs/API.md)
2. Include request/response examples
3. Document error codes
4. Add to API.md table of contents

```markdown
### New Endpoint

\`\`\`http
POST /endpoint
Authorization: Bearer <token>
\`\`\`

**Request**:
\`\`\`json
{ "field": "value" }
\`\`\`

**Response** (200 OK):
\`\`\`json
{ "success": true, "data": {} }
\`\`\`
```

### Update Architecture Documentation

When making architectural changes:

1. Update [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)
2. Update diagrams if needed
3. Document rationale in [docs/DECISIONS.md](./docs/DECISIONS.md)

---

## Performance Considerations

### Backend Performance

- Resume parsing: target < 2 seconds
- LLM analysis: target < 5 seconds
- Database queries: index critical paths
- Use caching: Redis for expensive operations

```typescript
// Good: Cache expensive LLM calls
async function getRecommendations(resumeId: string) {
  const cacheKey = `recommendations:${resumeId}`;
  const cached = await cache.get(cacheKey);
  if (cached) return cached;

  const recommendations = await llm.generate(resume);
  await cache.set(cacheKey, recommendations, 86400);
  return recommendations;
}
```

### Frontend Performance

- Code split components
- Lazy load routes
- Optimize images
- Monitor bundle size

```typescript
// Good: Lazy load heavy components
const AnalysisReport = lazy(() => import('./AnalysisReport'));

export function Dashboard() {
  return (
    <Suspense fallback={<Loading />}>
      <AnalysisReport />
    </Suspense>
  );
}
```

---

## Security Considerations

### Authentication & Authorization

- Never log passwords or tokens
- Validate all user inputs
- Check permissions before operations
- Use prepared queries (Prisma does this)

### Sensitive Data

- Encrypt PII at rest
- Use HTTPS only
- Don't expose errors to users
- Audit sensitive operations

```typescript
// Good: Validate and sanitize input
const createUser = async (email: string, password: string) => {
  // Validate format
  if (!isValidEmail(email)) throw new ValidationError();
  
  // Hash password
  const hash = await bcrypt.hash(password, 10);
  
  // Store securely
  return db.user.create({ email, passwordHash: hash });
};
```

---

## CI/CD Pipeline

### Automated Checks

Every push runs:
1. Lint check
2. Type check
3. Unit tests
4. Integration tests
5. Build

### Required Checks Before Merge

- ✅ All tests pass
- ✅ Coverage threshold met
- ✅ No lint errors
- ✅ Types check out
- ✅ Build succeeds

### Manual Checks (Code Review)

- Design consistency
- Code readability
- Test quality
- Documentation completeness

---

## Release Process

### Version Numbering

Use semantic versioning: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes

### Release Checklist

- [ ] Update version: `package.json`
- [ ] Update changelog: `CHANGELOG.md` (if exists)
- [ ] Create git tag: `git tag -a v1.0.0`
- [ ] Push tag: `git push origin v1.0.0`
- [ ] Build & deploy automatically via CI/CD

---

## Need Help?

- **Setup issues**: See [DEVELOPMENT.md](../DEVELOPMENT.md#troubleshooting)
- **Design questions**: Ask in team Slack
- **Code review feedback**: Clarify in PR comments
- **Feature ideas**: Create GitHub issue

---

## Resources

- [Development Setup](../DEVELOPMENT.md)
- [Documentation Index](./docs/README.md)
- [Architecture](./docs/ARCHITECTURE.md)
- [API Reference](./docs/API.md)

---

Thank you for contributing! 🎉
