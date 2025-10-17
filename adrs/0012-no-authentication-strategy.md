# ADR-0012: No Authentication Strategy

**Status**: Accepted
**Date**: 2025-10-16

## Context

CV Parser is a simple utility tool. Need to decide if authentication is required.

## Decision

**No authentication system** for MVP.

**Rationale**:
- Public utility tool (like paste bins)
- Stateless = no user accounts needed
- Privacy-first = no data to protect after processing
- Simplicity = faster time-to-market
- Rate limiting by IP sufficient for abuse prevention

## Consequences

✅ **Frictionless UX**: No signup/login required
✅ **Privacy**: No user tracking or accounts
✅ **Simplicity**: No auth complexity (sessions, JWTs, etc.)
✅ **Fast Onboarding**: Users can start immediately
❌ **No User History**: Can't save parsing history
❌ **IP-Based Rate Limiting Only**: Easier to abuse
❌ **No Personalization**: Can't store preferences

## Future Consideration

**If authentication added later**:
1. Optional feature (not required)
2. OAuth2 (Google, GitHub) for simplicity
3. Database required (ADR-0002 would be revisited)
4. Use Passport.js or NextAuth.js

## Security Without Auth

1. **Rate Limiting**: 10 requests/min per IP
2. **File Validation**: Prevent malicious uploads
3. **No Data Retention**: Nothing to steal
4. **CORS**: Restrict cross-origin requests
5. **Security Headers**: Helmet.js protection

## References

- SECURITY.md § No Authentication
- BUILD_SPEC.md § Authentication: Not required

**Related ADRs**: ADR-0002 (Stateless Architecture)
