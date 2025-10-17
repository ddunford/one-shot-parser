# ADR-0008: File Upload Handling (Multer)

**Status**: Accepted
**Date**: 2025-10-16

## Context

Need secure file upload handling with validation (type, size).

## Decision

Use **Multer** middleware with in-memory storage.

**Configuration**:
- Storage: `multer.memoryStorage()` (no disk writes)
- Limits: 5MB max, 1 file per request
- Validation: MIME type filter (PDF, DOCX only)

## Consequences

✅ **Security**: Server-side validation, no disk access
✅ **Privacy**: Files never touch disk
✅ **Simple**: Standard Express middleware
✅ **Performance**: In-memory faster than disk
❌ **Memory Usage**: Large files consume RAM
❌ **No Streaming**: Entire file in memory

## Implementation

```typescript
const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 5 * 1024 * 1024, files: 1 },
  fileFilter: (req, file, cb) => {
    const allowed = ['application/pdf', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document'];
    cb(null, allowed.includes(file.mimetype));
  }
});
```

## References

- SECURITY.md § File Upload Security
- API.md § POST /api/parse

**Related ADRs**: ADR-0003 (Document Processing)
