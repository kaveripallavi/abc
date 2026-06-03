# API Reference
## AI Resume Analyzer

**Version**: 1.0  
**Base URL**: `https://api.resumeanalyzer.com/v1`  
**Authentication**: Bearer token (JWT)

---

## Authentication Endpoints

### Register User
```http
POST /auth/register
```

**Request**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "userType": "job_seeker" // or "recruiter"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "id": "user-123",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "userType": "job_seeker",
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc...",
    "createdAt": "2026-06-03T10:00:00Z"
  }
}
```

---

### Login
```http
POST /auth/login
```

**Request**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user-123",
      "email": "user@example.com",
      "firstName": "John",
      "userType": "job_seeker"
    },
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc...",
    "expiresIn": 3600
  }
}
```

---

### Refresh Token
```http
POST /auth/refresh
```

**Request**:
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc...",
    "expiresIn": 3600
  }
}
```

---

## Resume Endpoints

### Upload Resume
```http
POST /resumes/upload
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Request**:
```
file: <binary PDF/DOCX/TXT>
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "id": "resume-123",
    "fileName": "john_doe_resume.pdf",
    "fileType": "pdf",
    "fileSize": 125430,
    "status": "pending", // pending, parsing, completed, failed
    "uploadedAt": "2026-06-03T10:00:00Z",
    "createdAt": "2026-06-03T10:00:00Z"
  }
}
```

---

### List Resumes
```http
GET /resumes
Authorization: Bearer <token>
```

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `status`: Filter by status (pending, parsing, completed, failed)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "resumes": [
      {
        "id": "resume-123",
        "fileName": "john_doe_resume.pdf",
        "fileType": "pdf",
        "status": "completed",
        "uploadedAt": "2026-06-03T10:00:00Z",
        "analysisCount": 2
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 5,
      "totalPages": 1
    }
  }
}
```

---

### Get Resume Details
```http
GET /resumes/{id}
Authorization: Bearer <token>
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "resume-123",
    "fileName": "john_doe_resume.pdf",
    "fileType": "pdf",
    "fileSize": 125430,
    "status": "completed",
    "uploadedAt": "2026-06-03T10:00:00Z",
    "updatedAt": "2026-06-03T10:05:00Z",
    "parsedContent": {
      "contact": {
        "name": "John Doe",
        "email": "john@example.com",
        "phone": "+1-555-0100",
        "location": "New York, NY"
      },
      "summary": "Senior Software Engineer with 10 years experience...",
      "education": [
        {
          "school": "MIT",
          "degree": "BS",
          "field": "Computer Science",
          "graduationYear": 2014
        }
      ],
      "experience": [
        {
          "company": "Tech Corp",
          "title": "Senior Engineer",
          "startDate": "2020-01",
          "endDate": null,
          "description": "Led team of 5 engineers...",
          "skills": ["Python", "Kubernetes", "AWS"]
        }
      ],
      "skills": ["Python", "JavaScript", "AWS", "Kubernetes", "PostgreSQL"],
      "certifications": [
        {
          "name": "AWS Solutions Architect",
          "issuer": "Amazon",
          "date": "2021-06"
        }
      ]
    }
  }
}
```

---

### Delete Resume
```http
DELETE /resumes/{id}
Authorization: Bearer <token>
```

**Response** (204 No Content)

---

## Analysis Endpoints

### Request Quality Analysis
```http
POST /analysis/quality
Authorization: Bearer <token>
```

**Request**:
```json
{
  "resumeId": "resume-123"
}
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "analysisId": "analysis-456",
    "resumeId": "resume-123",
    "status": "processing",
    "createdAt": "2026-06-03T10:10:00Z"
  }
}
```

---

### Get Analysis Result
```http
GET /analysis/{id}
Authorization: Bearer <token>
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "analysis-456",
    "resumeId": "resume-123",
    "status": "completed",
    "scores": {
      "quality": 78,
      "professionalism": 85,
      "ats": 72
    },
    "feedback": {
      "structure": {
        "score": 80,
        "message": "Good structure with clear sections",
        "suggestions": ["Add a professional summary", "Consider adding certifications section"]
      },
      "clarity": {
        "score": 75,
        "message": "Generally clear but some sections could be more concise",
        "suggestions": ["Reduce job description length", "Use more action verbs"]
      },
      "content": {
        "score": 78,
        "message": "Good content but missing some details",
        "suggestions": ["Add metrics to achievements", "Include more technical skills"]
      }
    },
    "recommendations": [
      {
        "priority": "high",
        "category": "keywords",
        "suggestion": "Add industry keywords: 'Machine Learning', 'Data Pipeline', 'Microservices'",
        "example": "Implemented Machine Learning pipeline for data processing"
      },
      {
        "priority": "medium",
        "category": "structure",
        "suggestion": "Reorganize experience section chronologically",
        "example": null
      }
    ],
    "completedAt": "2026-06-03T10:15:00Z"
  }
}
```

---

### Request ATS Compatibility Check
```http
POST /analysis/ats-score
Authorization: Bearer <token>
```

**Request**:
```json
{
  "resumeId": "resume-123"
}
```

**Response** (202 Accepted): Similar to quality analysis

---

## Job Matching Endpoints

### Score Resume Against Job
```http
POST /matching/score
Authorization: Bearer <token>
```

**Request**:
```json
{
  "resumeId": "resume-123",
  "jobDescription": "Senior Software Engineer needed...",
  "jobRequirements": {
    "minimumYears": 5,
    "requiredSkills": ["Python", "Kubernetes", "AWS"],
    "preferredSkills": ["Go", "Docker Compose"],
    "education": "BS Computer Science",
    "certifications": []
  }
}
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "matchId": "match-789",
    "status": "processing",
    "createdAt": "2026-06-03T10:20:00Z"
  }
}
```

---

### Get Match Result
```http
GET /matching/{id}
Authorization: Bearer <token>
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "match-789",
    "resumeId": "resume-123",
    "status": "completed",
    "scores": {
      "overallMatch": 82,
      "skillMatch": 88,
      "experienceMatch": 78,
      "keywordMatch": 81
    },
    "skillAnalysis": {
      "matched": [
        { "skill": "Python", "proficiency": "advanced", "years": 8 },
        { "skill": "Kubernetes", "proficiency": "intermediate", "years": 3 },
        { "skill": "AWS", "proficiency": "advanced", "years": 5 }
      ],
      "missing": [
        { "skill": "Go", "priority": "nice_to_have" },
        { "skill": "Docker Compose", "priority": "nice_to_have" }
      ]
    },
    "gapAnalysis": {
      "hardSkillGaps": [
        {
          "skill": "Go",
          "importance": "nice_to_have",
          "timeToLearn": 6,
          "learningPath": "Take Go fundamentals course + build projects"
        }
      ],
      "softSkillGaps": [],
      "experienceGaps": [],
      "keywordMissing": ["DevOps", "Agile"]
    },
    "recommendations": [
      {
        "category": "keywords",
        "suggestion": "Add DevOps keywords to highlight operational experience",
        "proposedText": "Led DevOps initiatives improving deployment time by 60%"
      },
      {
        "category": "skills",
        "suggestion": "Highlight Kubernetes expertise more prominently",
        "proposedText": "Architected Kubernetes cluster supporting 50+ microservices"
      }
    ],
    "completedAt": "2026-06-03T10:25:00Z"
  }
}
```

---

## Skill Gap Endpoints

### Analyze Skill Gaps
```http
POST /skills/gap-analysis
Authorization: Bearer <token>
```

**Request**:
```json
{
  "resumeId": "resume-123",
  "targetRole": "Senior Backend Engineer",
  "industry": "Technology"
}
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "gapAnalysisId": "gap-101",
    "status": "processing"
  }
}
```

---

### Get Skill Gap Result
```http
GET /skills/gap-analysis/{id}
Authorization: Bearer <token>
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "gap-101",
    "targetRole": "Senior Backend Engineer",
    "currentSkills": [
      { "skill": "Python", "proficiency": "advanced", "yearsOfExperience": 8 },
      { "skill": "AWS", "proficiency": "advanced", "yearsOfExperience": 5 }
    ],
    "marketSkills": [
      { "skill": "Go", "proficiency": "advanced", "demand": "high", "averageYears": 4 },
      { "skill": "Kubernetes", "proficiency": "advanced", "demand": "high", "averageYears": 3 },
      { "skill": "gRPC", "proficiency": "intermediate", "demand": "medium", "averageYears": 2 }
    ],
    "skillGaps": [
      {
        "skill": "Go",
        "currentProficiency": null,
        "requiredProficiency": "advanced",
        "priority": "high",
        "timeToLearn": 6,
        "marketDemand": "high",
        "recommendations": [
          {
            "type": "course",
            "name": "The Complete Go Developer Course",
            "provider": "Udemy",
            "duration": 20,
            "estimatedCost": 50,
            "url": "https://..."
          }
        ]
      },
      {
        "skill": "Kubernetes",
        "currentProficiency": "intermediate",
        "requiredProficiency": "advanced",
        "priority": "high",
        "timeToLearn": 4,
        "marketDemand": "high",
        "recommendations": []
      }
    ],
    "learningRoadmap": {
      "timelineMonths": 12,
      "phases": [
        {
          "phase": 1,
          "duration": 2,
          "focus": "Go fundamentals",
          "skills": ["Go basics", "Concurrency", "Testing"]
        },
        {
          "phase": 2,
          "duration": 2,
          "focus": "Advanced Kubernetes",
          "skills": ["Custom resources", "Operators", "Performance tuning"]
        }
      ]
    },
    "completedAt": "2026-06-03T10:30:00Z"
  }
}
```

---

## Recruiter Endpoints

### Upload Resume Batch
```http
POST /recruiter/batch-upload
Authorization: Bearer <token>
```

**Request**:
```
resumes: <file 1>, <file 2>, ... (multipart)
jobDescription: "Senior Engineer job description..."
batchName: "Q2 2026 Hiring Round"
```

**Response** (202 Accepted):
```json
{
  "success": true,
  "data": {
    "batchId": "batch-111",
    "batchName": "Q2 2026 Hiring Round",
    "totalResumes": 50,
    "status": "queued",
    "createdAt": "2026-06-03T10:35:00Z"
  }
}
```

---

### Get Batch Status
```http
GET /recruiter/batches/{id}
Authorization: Bearer <token>
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": "batch-111",
    "batchName": "Q2 2026 Hiring Round",
    "totalResumes": 50,
    "processedCount": 35,
    "failedCount": 2,
    "status": "processing",
    "progress": 70,
    "estimatedTimeRemaining": 5,
    "createdAt": "2026-06-03T10:35:00Z"
  }
}
```

---

### Get Batch Results & Rankings
```http
GET /recruiter/batches/{id}/results
Authorization: Bearer <token>
```

**Query Parameters**:
- `sortBy`: match_score, skill_match, experience_match (default: match_score)
- `limit`: Top N results (default: 20)
- `minScore`: Minimum match score filter (0-100)

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "batchId": "batch-111",
    "candidates": [
      {
        "rank": 1,
        "resumeId": "resume-abc",
        "candidateName": "Alice Smith",
        "email": "alice@example.com",
        "scores": {
          "overallMatch": 95,
          "skillMatch": 96,
          "experienceMatch": 94
        },
        "matchedSkills": ["Python", "Kubernetes", "AWS"],
        "missingSkills": ["Go"],
        "yearsOfExperience": 12,
        "currentRole": "Staff Engineer"
      },
      {
        "rank": 2,
        "resumeId": "resume-def",
        "candidateName": "Bob Johnson",
        "email": "bob@example.com",
        "scores": {
          "overallMatch": 88,
          "skillMatch": 90,
          "experienceMatch": 85
        },
        "matchedSkills": ["Python", "AWS"],
        "missingSkills": ["Kubernetes", "Go"],
        "yearsOfExperience": 8,
        "currentRole": "Senior Engineer"
      }
    ],
    "completedAt": "2026-06-03T10:45:00Z"
  }
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": [
      {
        "field": "fieldName",
        "message": "Specific field error"
      }
    ]
  },
  "timestamp": "2026-06-03T10:00:00Z",
  "requestId": "req-123456"
}
```

### Common Error Codes

| Code | Status | Description |
|------|--------|-------------|
| INVALID_REQUEST | 400 | Malformed request |
| VALIDATION_ERROR | 400 | Input validation failed |
| UNAUTHORIZED | 401 | Missing/invalid token |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |

---

## Rate Limiting

All endpoints are rate-limited:

- **Default**: 100 requests/minute per user
- **Authentication endpoints**: 10 requests/minute per IP
- **Batch upload**: 5 requests/minute per user

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1686053000
```

---

## Pagination

List endpoints support pagination:

**Query Parameters**:
- `page`: Page number (default: 1, min: 1)
- `limit`: Items per page (default: 20, max: 100)

**Response includes**:
```json
{
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

## Webhooks (Future)

Batch completion webhooks (Phase 3):

```http
POST {webhook_url}
Content-Type: application/json
X-Webhook-Signature: sha256=...
```

**Payload**:
```json
{
  "event": "batch.completed",
  "batchId": "batch-111",
  "status": "completed",
  "totalResumes": 50,
  "resultsUrl": "https://api.resumeanalyzer.com/v1/recruiter/batches/batch-111/results"
}
```
