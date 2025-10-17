# Testing Strategy - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## Testing Philosophy

**Goal**: Achieve ≥80% code coverage with a balanced testing pyramid

**Principles**:
1. **Fast Feedback**: Unit tests run in <10s
2. **Confidence**: Integration tests cover critical paths
3. **Realism**: E2E tests validate real user flows
4. **Maintainability**: Tests are readable and don't break easily

---

## Testing Pyramid

```
        /\
       /  \
      / E2E \  ← 10% (UI flows, real LLMs)
     /______\
    /        \
   /   Int    \ ← 20% (Component interactions)
  /__________\
 /            \
/     Unit     \ ← 70% (Business logic, adapters)
/______________\
```

**Target Distribution**:
- **70% Unit Tests**: Fast, isolated, comprehensive
- **20% Integration Tests**: Component interactions, mocked LLMs
- **10% E2E Tests**: Full stack, real browser, real/mocked LLMs

**Coverage Target**: ≥80% overall, ≥90% for critical paths

---

## Unit Testing (70%)

### Backend Unit Tests

**Framework**: Jest

**What to Test**:
- Document extractors (PDF, DOCX)
- LLM adapters (mocked API calls)
- Data validators
- Error handlers
- Utility functions

**Example Tests**:

```typescript
// src/backend/tests/unit/extractors/PDFExtractor.test.ts
describe('PDFExtractor', () => {
  let extractor: PDFExtractor;

  beforeEach(() => {
    extractor = new PDFExtractor();
  });

  test('extracts text from valid PDF', async () => {
    const buffer = fs.readFileSync('fixtures/sample-cv.pdf');
    const result = await extractor.extract(buffer);

    expect(result.text).toContain('John Doe');
    expect(result.text).toContain('Software Engineer');
    expect(result.pageCount).toBe(2);
    expect(result.wordCount).toBeGreaterThan(100);
  });

  test('handles corrupted PDF gracefully', async () => {
    const buffer = Buffer.from('not a pdf');

    await expect(extractor.extract(buffer))
      .rejects
      .toThrow('Unable to extract text');
  });

  test('respects timeout for large PDFs', async () => {
    const largeBuffer = fs.readFileSync('fixtures/large-cv.pdf');

    await expect(extractor.extract(largeBuffer, { timeout: 100 }))
      .rejects
      .toThrow('Extraction timeout');
  }, 10000);
});
```

```typescript
// src/backend/tests/unit/llm/OllamaAdapter.test.ts
describe('OllamaAdapter', () => {
  let adapter: OllamaAdapter;
  let mockFetch: jest.Mock;

  beforeEach(() => {
    mockFetch = jest.fn();
    global.fetch = mockFetch;
    adapter = new OllamaAdapter({
      baseUrl: 'http://localhost:11434',
      model: 'llama3.1'
    });
  });

  test('parses CV successfully', async () => {
    const mockResponse = {
      message: {
        content: JSON.stringify({
          personalInfo: { name: 'John Doe', email: 'john@example.com' },
          skills: { technical: ['JavaScript'], soft: [], languages: [], certifications: [] },
          education: [],
          experience: []
        })
      }
    };

    mockFetch.mockResolvedValue({
      ok: true,
      json: async () => mockResponse
    });

    const result = await adapter.parseCV('Sample CV text...');

    expect(result.personalInfo.name).toBe('John Doe');
    expect(mockFetch).toHaveBeenCalledWith(
      'http://localhost:11434/api/chat',
      expect.objectContaining({
        method: 'POST',
        headers: { 'Content-Type': 'application/json' }
      })
    );
  });

  test('handles LLM timeout', async () => {
    mockFetch.mockImplementation(() =>
      new Promise((resolve) => setTimeout(resolve, 70000))
    );

    await expect(adapter.parseCV('text', { timeout: 1000 }))
      .rejects
      .toThrow('timeout');
  });

  test('handles invalid JSON response', async () => {
    mockFetch.mockResolvedValue({
      ok: true,
      json: async () => ({ message: { content: 'not json' } })
    });

    await expect(adapter.parseCV('text'))
      .rejects
      .toThrow('Invalid JSON');
  });
});
```

**Run Command**:
```bash
npm test -- --coverage --testPathPattern=unit
```

---

### Frontend Unit Tests

**Framework**: Vitest + React Testing Library

**What to Test**:
- Component rendering
- User interactions
- State management
- API client functions
- Custom hooks

**Example Tests**:

```typescript
// src/frontend/tests/unit/FileUpload.test.tsx
describe('FileUpload', () => {
  test('renders upload area', () => {
    render(<FileUpload onFileSelected={jest.fn()} />);

    expect(screen.getByText(/drag.*drop/i)).toBeInTheDocument();
    expect(screen.getByText(/click to browse/i)).toBeInTheDocument();
  });

  test('calls onFileSelected when file is selected', async () => {
    const onFileSelected = jest.fn();
    render(<FileUpload onFileSelected={onFileSelected} />);

    const file = new File(['content'], 'cv.pdf', { type: 'application/pdf' });
    const input = screen.getByLabelText(/upload/i);

    await userEvent.upload(input, file);

    expect(onFileSelected).toHaveBeenCalledWith(file);
  });

  test('shows error for invalid file type', async () => {
    render(<FileUpload onFileSelected={jest.fn()} />);

    const file = new File(['content'], 'cv.txt', { type: 'text/plain' });
    const input = screen.getByLabelText(/upload/i);

    await userEvent.upload(input, file);

    expect(screen.getByText(/unsupported file format/i)).toBeInTheDocument();
  });

  test('shows error for file too large', async () => {
    render(<FileUpload onFileSelected={jest.fn()} maxSize={1024} />);

    const largeFile = new File(['a'.repeat(2000)], 'cv.pdf', { type: 'application/pdf' });
    const input = screen.getByLabelText(/upload/i);

    await userEvent.upload(input, largeFile);

    expect(screen.getByText(/file size exceeds/i)).toBeInTheDocument();
  });
});
```

**Run Command**:
```bash
npm test -- --run --coverage
```

---

## Integration Testing (20%)

### Backend Integration Tests

**Framework**: Jest + Supertest

**What to Test**:
- Full API endpoints
- Middleware chain
- Error handling
- File upload pipeline
- LLM adapter integration (mocked)

**Example Tests**:

```typescript
// src/backend/tests/integration/api.test.ts
describe('POST /api/parse', () => {
  let app: Express;

  beforeAll(() => {
    app = createApp();
  });

  test('successfully parses PDF CV', async () => {
    const response = await request(app)
      .post('/api/parse')
      .attach('file', 'fixtures/sample-cv.pdf')
      .field('provider', 'ollama');

    expect(response.status).toBe(200);
    expect(response.body.success).toBe(true);
    expect(response.body.data.personalInfo.name).toBeDefined();
    expect(response.body.meta.processingTime).toBeGreaterThan(0);
  });

  test('rejects invalid file type', async () => {
    const response = await request(app)
      .post('/api/parse')
      .attach('file', Buffer.from('text'), 'cv.txt');

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('INVALID_FILE_TYPE');
  });

  test('handles missing file', async () => {
    const response = await request(app)
      .post('/api/parse')
      .field('provider', 'ollama');

    expect(response.status).toBe(400);
    expect(response.body.error.code).toBe('MISSING_FILE');
  });

  test('enforces rate limit', async () => {
    // Make 11 requests
    const requests = Array(11).fill(null).map((_, i) =>
      request(app)
        .post('/api/parse')
        .attach('file', 'fixtures/sample-cv.pdf')
    );

    const responses = await Promise.all(requests);
    const tooManyRequests = responses.filter(r => r.status === 429);

    expect(tooManyRequests.length).toBeGreaterThan(0);
  });
});
```

**Run Command**:
```bash
npm test -- --testPathPattern=integration
```

---

## E2E Testing (10%)

### Frontend E2E Tests

**Framework**: Playwright

**🚨 CRITICAL: E2E Test URL Configuration**

**E2E tests MUST run against the primary development URL**: `https://cvparse.demosrv.uk`

**Why?**
- Validates the **real user experience** through the Traefik reverse proxy
- Tests HTTPS routing, SSL certificates, and proxy configuration
- Ensures tests match documented user journeys
- Detects issues that won't appear when bypassing Traefik

**Configuration**:
```typescript
// src/frontend/playwright.config.ts
export default defineConfig({
  use: {
    baseURL: process.env.PLAYWRIGHT_BASE_URL || 'https://cvparse.demosrv.uk',
  }
});
```

```yaml
# docker-compose.yml
services:
  frontend:
    environment:
      - PLAYWRIGHT_BASE_URL=https://cvparse.demosrv.uk
```

**❌ NEVER use internal Docker URLs** (`http://frontend:3000`, `http://localhost:5173`) for E2E tests - they bypass Traefik and don't reflect real user experience.

**What to Test**:
- Complete user journeys
- Real browser interactions
- Cross-browser compatibility
- Error flows

**Example Tests**:

```typescript
// tests/e2e/cv-parsing.spec.ts
import { test, expect } from '@playwright/test';

test.describe('CV Parsing Flow', () => {
  test('uploads and parses PDF CV successfully', async ({ page }) => {
    await page.goto('https://cvparse.demosrv.uk');

    // Upload file
    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles('fixtures/sample-cv.pdf');

    // Verify file preview
    await expect(page.locator('text=sample-cv.pdf')).toBeVisible();

    // Select provider
    await page.selectOption('select[name="provider"]', 'ollama');

    // Submit
    await page.click('button:has-text("Parse CV")');

    // Wait for processing
    await expect(page.locator('text=Analyzing with')).toBeVisible();

    // Wait for results (max 60s)
    await expect(page.locator('text=Personal Information')).toBeVisible({ timeout: 60000 });

    // Verify sections
    await expect(page.locator('text=Skills')).toBeVisible();
    await expect(page.locator('text=Education')).toBeVisible();
    await expect(page.locator('text=Experience')).toBeVisible();

    // Verify data extracted
    const personalInfo = page.locator('[data-testid="personal-info"]');
    await expect(personalInfo).toContainText('John Doe');
    await expect(personalInfo).toContainText('john@example.com');

    // Test JSON export
    await page.click('button:has-text("View JSON")');
    await expect(page.locator('pre')).toBeVisible();

    await page.click('button:has-text("Copy to Clipboard")');
    await expect(page.locator('text=Copied')).toBeVisible();
  });

  test('handles invalid file type error', async ({ page }) => {
    await page.goto('https://cvparse.demosrv.uk');

    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles('fixtures/invalid.txt');

    await expect(page.locator('text=Unsupported file format')).toBeVisible();
  });

  test('handles LLM timeout gracefully', async ({ page }) => {
    // Mock slow LLM response
    await page.route('**/api/parse', async (route) => {
      await new Promise(resolve => setTimeout(resolve, 65000));
      await route.continue();
    });

    await page.goto('https://cvparse.demosrv.uk');

    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles('fixtures/sample-cv.pdf');

    await page.click('button:has-text("Parse CV")');

    await expect(page.locator('text=timed out')).toBeVisible({ timeout: 70000 });
    await expect(page.locator('button:has-text("Retry")').toBeVisible();
  });
});

test.describe('Provider Switching', () => {
  test('switches between providers', async ({ page }) => {
    await page.goto('https://cvparse.demosrv.uk');

    const fileInput = page.locator('input[type="file"]');
    await fileInput.setInputFiles('fixtures/sample-cv.pdf');

    // Test with Ollama
    await page.selectOption('select[name="provider"]', 'ollama');
    await page.click('button:has-text("Parse CV")');
    await expect(page.locator('text=Personal Information')).toBeVisible({ timeout: 60000 });

    const ollamaResult = await page.locator('[data-testid="personal-info"]').textContent();

    // Re-parse with OpenAI
    await page.click('button:has-text("Parse Again")');
    await fileInput.setInputFiles('fixtures/sample-cv.pdf');
    await page.selectOption('select[name="provider"]', 'openai');
    await page.click('button:has-text("Parse CV")');
    await expect(page.locator('text=Personal Information')).toBeVisible({ timeout: 60000 });

    const openaiResult = await page.locator('[data-testid="personal-info"]').textContent();

    // Results should be similar but not identical
    expect(ollamaResult).toBeTruthy();
    expect(openaiResult).toBeTruthy();
  });
});
```

**Run Command**:
```bash
npx playwright test
```

**Browsers**: Chromium, Firefox, WebKit

---

## Test Data & Fixtures

### Sample CVs

**fixtures/sample-cv.pdf**: Standard 2-page CV (software engineer)
**fixtures/sample-cv.docx**: Same CV in Word format
**fixtures/minimal-cv.pdf**: Very short CV (1 page, basic info only)
**fixtures/complex-cv.pdf**: Multi-page with tables, images (10 pages)
**fixtures/non-english-cv.pdf**: CV in Spanish/French
**fixtures/corrupted.pdf**: Invalid PDF file
**fixtures/large-cv.pdf**: 10MB file (exceeds limit)

### Mock LLM Responses

```typescript
// tests/mocks/llm-responses.ts
export const mockCVData = {
  personalInfo: {
    name: 'John Doe',
    email: 'john.doe@example.com',
    phone: '+1-555-123-4567',
    address: 'San Francisco, CA'
  },
  skills: {
    technical: ['JavaScript', 'TypeScript', 'React', 'Node.js'],
    soft: ['Leadership', 'Communication'],
    languages: ['English (Native)'],
    certifications: []
  },
  education: [{
    institution: 'Stanford University',
    degree: 'BS',
    field: 'Computer Science',
    startDate: '2010-09',
    endDate: '2014-06'
  }],
  experience: [{
    company: 'Tech Corp',
    title: 'Software Engineer',
    startDate: '2014-07',
    endDate: null,
    current: true,
    responsibilities: ['Built features', 'Led team'],
    achievements: ['Improved performance by 50%']
  }],
  recommendations: {
    jobTitles: ['Senior Software Engineer', 'Tech Lead'],
    reasoning: 'Based on 10 years of experience'
  }
};
```

---

## Coverage Requirements

**Overall**: ≥80%

**Critical Components**: ≥90%
- ParseController
- LLM Adapters
- Document Extractors
- Data Validators

**Nice-to-Have Components**: ≥60%
- UI components
- Utility functions

**Exclude from Coverage**:
- Configuration files
- Type definitions
- Test files

**Coverage Report**:
```bash
npm run test:coverage
open coverage/index.html
```

---

## CI/CD Integration

### GitHub Actions Workflow

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Run integration tests
        run: npm run test:integration

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Start services
        run: docker compose up -d

      - name: Wait for services
        run: npx wait-on https://cvparse.demosrv.uk

      - name: Run E2E tests
        run: npx playwright test

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Performance Testing

### Load Testing (Future)

**Tool**: k6

**Scenario**: Simulate 50 concurrent users uploading CVs

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 50,        // 50 virtual users
  duration: '5m'  // 5 minutes
};

export default function () {
  const file = open('sample-cv.pdf', 'b');
  const data = {
    file: http.file(file, 'cv.pdf'),
    provider: 'ollama'
  };

  const res = http.post('https://cvparse.demosrv.uk/api/parse', data);

  check(res, {
    'status is 200': (r) => r.status === 200,
    'processing time < 60s': (r) => r.json('meta.processingTime') < 60000
  });
}
```

---

## Test Maintenance

### When to Update Tests

1. **New Feature**: Write tests first (TDD)
2. **Bug Fix**: Add regression test
3. **Refactoring**: Ensure tests still pass
4. **Breaking Change**: Update affected tests

### Test Review Checklist

- [ ] Tests are readable and self-explanatory
- [ ] Tests are isolated (no interdependencies)
- [ ] Tests are deterministic (no flakiness)
- [ ] Fixtures are realistic
- [ ] Mocks are minimal and necessary
- [ ] Coverage meets target (≥80%)

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 2
**Owner**: Engineering/QA Team
