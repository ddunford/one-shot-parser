# ADR-0010: Error Handling in Stateless System

**Status**: Accepted
**Date**: 2025-10-16

## Context

Stateless architecture means no retry mechanism with stored state. Errors must be handled immediately within request lifecycle.

## Decision

**Fail-fast with clear error messages** and client-side retry.

**Error Strategy**:
1. Validate early (client + server)
2. Timeout long-running requests (60s)
3. Return specific error codes and actionable messages
4. Log errors (metadata only, no PII)
5. Graceful degradation (partial results if possible)
6. Client handles retry logic

## Consequences

✅ **Clear Feedback**: Users know exactly what went wrong
✅ **Fast Failure**: Don't waste time on doomed requests
✅ **Actionable**: Error messages suggest solutions
✅ **Privacy**: No error state stored server-side
❌ **No Automatic Retry**: User must manually retry
❌ **Lost Progress**: Timeouts discard all progress

## Error Types

- `INVALID_FILE_TYPE` (400): Client validation failed
- `FILE_TOO_LARGE` (400): Exceeds 5MB
- `EXTRACTION_FAILED` (422): Can't extract text
- `LLM_TIMEOUT` (504): Processing >60s
- `PROVIDER_UNAVAILABLE` (503): Can't connect to LLM
- `RATE_LIMIT_EXCEEDED` (429): Too many requests

## References

- API.md § Error Handling
- FUNCTIONAL.md § Feature 9 (Error Handling)

**Related ADRs**: ADR-0002 (Stateless Architecture)
