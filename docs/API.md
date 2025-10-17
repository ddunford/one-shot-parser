# API Specification - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## API Overview

The CV Parser API provides a simple REST interface for uploading and parsing CV documents. The API is stateless, requiring no authentication, and supports multiple LLM providers for parsing.

**Base URL**: `https://cvparse.demosrv.uk/api`

**Protocol**: HTTPS only

**Data Format**: JSON (except file upload endpoint which uses multipart/form-data)

---

## Authentication

**Auth Required**: No

This is a public, open API with no authentication mechanism. Rate limiting is applied per IP address to prevent abuse.

---

## Rate Limiting

**Limit**: 10 requests per minute per IP address

**Headers**:
```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1634567890
```

**Response (429 Too Many Requests)**:
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please wait 60 seconds before retrying.",
    "retryAfter": 45
  }
}
```

---

## Endpoints

### 1. Parse CV

Upload and parse a CV document.

**Endpoint**: `POST /api/parse`

**Content-Type**: `multipart/form-data`

**Request Parameters**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | File | Yes | CV document (PDF or DOCX, max 5MB) |
| `provider` | string | No | LLM provider: `ollama` (default), `openai`, `bedrock` |

**Example Request**:
```bash
curl -X POST https://cvparse.demosrv.uk/api/parse \
  -F "file=@/path/to/cv.pdf" \
  -F "provider=ollama"
```

**Success Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "personalInfo": {
      "name": "John Doe",
      "email": "john.doe@email.com",
      "phone": "+1-555-123-4567",
      "address": "San Francisco, CA",
      "linkedin": "https://linkedin.com/in/johndoe",
      "github": "https://github.com/johndoe"
    },
    "summary": "Experienced software engineer...",
    "skills": {
      "technical": ["JavaScript", "Python", "Docker", "AWS"],
      "soft": ["Leadership", "Communication", "Problem Solving"],
      "languages": ["English (Native)", "Spanish (Fluent)"],
      "certifications": ["AWS Certified Solutions Architect"]
    },
    "education": [
      {
        "institution": "Stanford University",
        "degree": "Bachelor of Science",
        "field": "Computer Science",
        "startDate": "2010-09",
        "endDate": "2014-06",
        "gpa": "3.8"
      }
    ],
    "experience": [
      {
        "company": "Tech Corp",
        "title": "Senior Software Engineer",
        "location": "San Francisco, CA",
        "startDate": "2018-01",
        "endDate": null,
        "current": true,
        "responsibilities": [
          "Led team of 5 engineers",
          "Architected microservices platform"
        ],
        "achievements": [
          "Reduced deployment time by 60%",
          "Improved system reliability to 99.99%"
        ]
      }
    ],
    "recommendations": {
      "jobTitles": [
        "Engineering Manager",
        "Senior Software Engineer",
        "Technical Lead"
      ],
      "reasoning": "Based on leadership experience and technical skills"
    }
  },
  "meta": {
    "processingTime": 32450,
    "provider": "ollama",
    "stages": {
      "extraction": 3200,
      "llmProcessing": 28000,
      "validation": 1250
    }
  }
}
```

**Error Responses**:

**400 Bad Request - Invalid File Type**:
```json
{
  "success": false,
  "error": {
    "code": "INVALID_FILE_TYPE",
    "message": "Unsupported file format. Please upload a PDF or DOCX file.",
    "details": {
      "receivedType": "text/plain",
      "supportedTypes": ["application/pdf", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"]
    }
  }
}
```

**400 Bad Request - File Too Large**:
```json
{
  "success": false,
  "error": {
    "code": "FILE_TOO_LARGE",
    "message": "File size exceeds 5MB limit.",
    "details": {
      "fileSize": 7340032,
      "maxSize": 5242880
    }
  }
}
```

**400 Bad Request - Missing File**:
```json
{
  "success": false,
  "error": {
    "code": "MISSING_FILE",
    "message": "No file was uploaded. Please include a 'file' parameter."
  }
}
```

**422 Unprocessable Entity - Extraction Failed**:
```json
{
  "success": false,
  "error": {
    "code": "EXTRACTION_FAILED",
    "message": "Unable to extract text from document. The file may be corrupted or password-protected.",
    "details": {
      "stage": "extraction"
    }
  }
}
```

**500 Internal Server Error - LLM Processing Failed**:
```json
{
  "success": false,
  "error": {
    "code": "LLM_PROCESSING_FAILED",
    "message": "AI processing failed. Please try again or use a different provider.",
    "details": {
      "provider": "ollama",
      "stage": "llmProcessing"
    }
  }
}
```

**504 Gateway Timeout - LLM Timeout**:
```json
{
  "success": false,
  "error": {
    "code": "LLM_TIMEOUT",
    "message": "Processing timed out after 60 seconds. Try using a faster AI provider (OpenAI) or a simpler CV.",
    "details": {
      "provider": "ollama",
      "timeout": 60000
    }
  }
}
```

**503 Service Unavailable - Provider Unavailable**:
```json
{
  "success": false,
  "error": {
    "code": "PROVIDER_UNAVAILABLE",
    "message": "Cannot connect to Ollama at http://inference.lan:11434. Ensure Ollama is running.",
    "details": {
      "provider": "ollama",
      "endpoint": "http://inference.lan:11434"
    }
  }
}
```

---

### 2. Get Provider Status

Check the availability and status of LLM providers.

**Endpoint**: `GET /api/providers`

**Example Request**:
```bash
curl https://cvparse.demosrv.uk/api/providers
```

**Success Response (200 OK)**:
```json
{
  "providers": [
    {
      "id": "ollama",
      "name": "Ollama (Local)",
      "status": "online",
      "latency": 120,
      "endpoint": "http://inference.lan:11434",
      "estimatedTime": "20-60 seconds"
    },
    {
      "id": "openai",
      "name": "OpenAI GPT-4",
      "status": "unconfigured",
      "estimatedTime": "10-20 seconds"
    },
    {
      "id": "bedrock",
      "name": "AWS Bedrock (Claude)",
      "status": "unconfigured",
      "estimatedTime": "15-30 seconds"
    }
  ]
}
```

**Status Values**:
- `online`: Provider is available and responding
- `offline`: Provider is configured but not responding
- `unconfigured`: Provider requires API key/configuration

---

### 3. Health Check

Check API health and dependencies.

**Endpoint**: `GET /api/health`

**Example Request**:
```bash
curl https://cvparse.demosrv.uk/api/health
```

**Success Response (200 OK)**:
```json
{
  "status": "healthy",
  "timestamp": "2025-10-16T14:30:00Z",
  "uptime": 86400,
  "version": "1.0.0",
  "providers": {
    "ollama": {
      "status": "online",
      "latency": 120
    },
    "openai": {
      "status": "unconfigured"
    },
    "bedrock": {
      "status": "unconfigured"
    }
  }
}
```

**Unhealthy Response (503 Service Unavailable)**:
```json
{
  "status": "unhealthy",
  "timestamp": "2025-10-16T14:30:00Z",
  "errors": [
    "All LLM providers are unavailable"
  ]
}
```

---

## Data Schema

### CVData Object

The main response data structure for parsed CV information.

**Type Definition** (TypeScript):
```typescript
interface CVData {
  personalInfo: PersonalInfo;
  summary?: string;
  skills: Skills;
  education: Education[];
  experience: Experience[];
  recommendations?: Recommendations;
}

interface PersonalInfo {
  name: string | null;
  email: string | null;
  phone: string | null;
  address: string | null;
  linkedin?: string | null;
  github?: string | null;
  website?: string | null;
}

interface Skills {
  technical: string[];
  soft: string[];
  languages: string[];
  certifications: string[];
}

interface Education {
  institution: string;
  degree: string;
  field: string;
  startDate: string | null;    // Format: YYYY-MM
  endDate: string | null;       // Format: YYYY-MM or "Present"
  gpa?: string | null;
}

interface Experience {
  company: string;
  title: string;
  location?: string | null;
  startDate: string | null;     // Format: YYYY-MM
  endDate: string | null;        // Format: YYYY-MM or "Present" or null
  current: boolean;
  responsibilities: string[];
  achievements: string[];
}

interface Recommendations {
  jobTitles: string[];
  reasoning: string;
}
```

**JSON Schema**: See FUNCTIONAL.md for complete JSON Schema definition

---

## Error Handling

### Error Response Structure

All error responses follow this consistent structure:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      // Additional context (optional)
    }
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_FILE_TYPE` | 400 | Unsupported file format |
| `FILE_TOO_LARGE` | 400 | File exceeds 5MB limit |
| `MISSING_FILE` | 400 | No file in request |
| `INVALID_PROVIDER` | 400 | Unknown provider specified |
| `EXTRACTION_FAILED` | 422 | Text extraction error |
| `LLM_PROCESSING_FAILED` | 500 | LLM returned error |
| `LLM_TIMEOUT` | 504 | Processing exceeded 60s |
| `PROVIDER_UNAVAILABLE` | 503 | Cannot connect to provider |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## CORS Policy

**Allowed Origins**: `*` (all origins allowed for public API)

**Allowed Methods**: `GET, POST, OPTIONS`

**Allowed Headers**: `Content-Type, Accept`

**Credentials**: Not allowed (no authentication)

---

## Integration Examples

### JavaScript (Fetch API)

```javascript
async function parseCV(file, provider = 'ollama') {
  const formData = new FormData();
  formData.append('file', file);
  formData.append('provider', provider);

  try {
    const response = await fetch('https://cvparse.demosrv.uk/api/parse', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error.message);
    }

    const result = await response.json();
    return result.data;
  } catch (error) {
    console.error('CV parsing failed:', error);
    throw error;
  }
}

// Usage
const fileInput = document.querySelector('input[type="file"]');
fileInput.addEventListener('change', async (e) => {
  const file = e.target.files[0];
  const cvData = await parseCV(file, 'ollama');
  console.log('Parsed CV:', cvData);
});
```

### Python (requests)

```python
import requests

def parse_cv(file_path, provider='ollama'):
    url = 'https://cvparse.demosrv.uk/api/parse'

    with open(file_path, 'rb') as f:
        files = {'file': f}
        data = {'provider': provider}
        response = requests.post(url, files=files, data=data)

    response.raise_for_status()
    return response.json()['data']

# Usage
cv_data = parse_cv('/path/to/cv.pdf', provider='ollama')
print(f"Name: {cv_data['personalInfo']['name']}")
print(f"Skills: {cv_data['skills']['technical']}")
```

### cURL

```bash
# Basic parse with Ollama
curl -X POST https://cvparse.demosrv.uk/api/parse \
  -F "file=@cv.pdf" \
  -F "provider=ollama"

# Parse with OpenAI
curl -X POST https://cvparse.demosrv.uk/api/parse \
  -F "file=@resume.docx" \
  -F "provider=openai"

# Check provider status
curl https://cvparse.demosrv.uk/api/providers

# Health check
curl https://cvparse.demosrv.uk/api/health
```

---

## Performance Considerations

### Processing Times

| Provider | Typical Time | Max Time |
|----------|--------------|----------|
| Ollama | 20-60s | 60s (timeout) |
| OpenAI | 10-20s | 60s (timeout) |
| Bedrock | 15-30s | 60s (timeout) |

**Note**: Times vary based on CV complexity, model size, and server load.

### File Size Limits

- **Max Size**: 5MB
- **Recommended**: 1-2MB
- **Typical CV**: 500KB - 1MB

### Rate Limits

- **Per IP**: 10 requests/minute
- **Burst**: Not allowed
- **Backoff**: Linear (60 second wait after limit)

---

## Versioning

**Current Version**: 1.0

**Versioning Strategy**: Semantic Versioning (SemVer)

**Breaking Changes**: Will be announced with major version bump (v2.0)

**Backward Compatibility**: Maintained within major versions

**Deprecation Policy**: 6 months notice for deprecated features

---

## Support & Feedback

**Issues**: https://github.com/your-org/cv-parser/issues

**Documentation**: This file + FUNCTIONAL.md

**Status Page**: https://cvparse.demosrv.uk/api/health

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 1
**Owner**: Engineering Team
