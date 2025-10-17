# ADR-0002: Stateless No-Database Architecture

**Status**: Accepted
**Date**: 2025-10-16
**Decision Makers**: Engineering Team

---

## Context

CV documents contain sensitive PII (names, emails, phone numbers, addresses). We need to decide whether to persist parsed CV data in a database or operate statelessly.

**Key Considerations**:
1. Privacy regulations (GDPR, CCPA) require careful handling of PII
2. No user authentication required (public tool)
3. No need for historical parsing data
4. Scalability requirements (horizontal scaling preferred)
5. Deployment simplicity (minimize infrastructure)

---

## Decision

We will build a **completely stateless application with no database**.

**Implementation**:
- All file processing happens in-memory during request lifecycle
- CV files and extracted text are discarded after response
- No user accounts, sessions, or parsing history
- Application state exists only in Node.js process memory (~10-70 seconds per request)

---

## Consequences

### Positive

✅ **Privacy-First**: No data retention = no data breach risk
✅ **GDPR Compliant**: No PII storage = simplified compliance
✅ **Horizontal Scaling**: Stateless = trivial to scale (no shared state)
✅ **Simple Deployment**: No database migrations, backups, or maintenance
✅ **Lower Cost**: No database hosting fees
✅ **Fast Recovery**: Service restart has no data loss concerns

### Negative

❌ **No User History**: Users can't retrieve past parses
❌ **No Analytics**: Can't track individual user behavior
❌ **No Caching**: Can't cache results by file hash
❌ **No Authentication**: Can't implement user accounts without database

### Neutral

🔵 **Performance**: In-memory processing is fast but limited by RAM
🔵 **Rate Limiting**: IP-based only (can't track by user account)

---

## Alternatives Considered

### Alternative 1: PostgreSQL with Full Persistence

**Approach**: Store users, parsing history, and CV data

**Rejected Because**:
- Introduces privacy risks (PII storage)
- Requires GDPR compliance measures (encryption, right to erasure, etc.)
- Adds complexity (migrations, backups, scaling)
- Not needed for MVP use case

### Alternative 2: Redis Caching Layer

**Approach**: Cache parsed results by file hash for 1 hour

**Rejected for MVP, Possible Future Enhancement**:
- Adds infrastructure dependency
- Still requires careful PII handling
- Marginal benefit (most users parse once)
- Can be added later if caching proves valuable

### Alternative 3: MongoDB Document Store

**Approach**: Flexible schema for varying CV structures

**Rejected Because**:
- Same privacy concerns as PostgreSQL
- No clear advantage over PostgreSQL for this use case
- Adds technology complexity

---

## Implementation Notes

**Memory Management**:
```typescript
// File stored in memory only during request
app.post('/api/parse', upload.single('file'), async (req, res) => {
  const fileBuffer = req.file.buffer;  // In-memory
  const result = await parseService.parseCV(fileBuffer, provider);
  res.json(result);
  // fileBuffer garbage collected after response
});
```

**No Persistence**:
```typescript
// ✅ CORRECT: No database writes
async parseCV(file: Buffer): Promise<CVData> {
  const text = await extractor.extract(file);
  const cvData = await llmAdapter.parseCV(text);
  return cvData;  // Return to client, don't store
}
```

**Logging Policy**:
```typescript
// ✅ CORRECT: Log metadata only, no PII
logger.info('CV parsed', {
  fileSize: file.length,
  provider: 'ollama',
  processingTime: 32000,
  success: true
  // NO fileName, NO extractedText, NO cvData
});
```

---

## Future Considerations

**If Database Becomes Necessary**:

1. **User Accounts** → PostgreSQL with encrypted PII fields
2. **Parsing History** → Store file hash only, not content
3. **Analytics** → Aggregate metrics only, no individual tracking
4. **Caching** → Redis with 1-hour TTL, file hash as key

**Migration Path**:
- Add database as optional feature (feature flag)
- Existing stateless behavior remains default
- Users opt-in to persistence

---

## References

- DATABASE.md § Rationale for No Database
- SECURITY.md § Privacy & Data Protection
- ARCHITECTURE.md § Stateless Architecture

---

**Related ADRs**:
- ADR-0010: Error Handling in Stateless System
- ADR-0012: No Authentication Strategy
