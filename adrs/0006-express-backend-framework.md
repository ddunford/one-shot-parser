# ADR-0006: Express Backend Framework

**Status**: Accepted
**Date**: 2025-10-16

## Context

Need Node.js web framework for REST API with file uploads.

## Decision

Use **Express.js** - lightweight, mature, flexible.

## Rationale

✅ Minimal, unopinionated (perfect for simple API)
✅ Excellent middleware ecosystem (Multer, helmet, cors)
✅ Fast hot reload with nodemon
✅ TypeScript support
✅ Team familiarity

## Alternatives Considered

- **NestJS**: Over-engineered for stateless API
- **Fastify**: Learning curve, not necessary for this scale
- **Koa**: Less mature ecosystem

## Consequences

✅ **Rapid Development**: Minimal boilerplate
✅ **Flexibility**: Choose only needed middleware
✅ **Mature Ecosystem**: Solutions for common problems
❌ **Manual Structure**: Must organize code ourselves
❌ **No Built-in Validation**: Need separate library

## References

- ARCHITECTURE.md § Express Application

**Related ADRs**: ADR-0008 (File Upload Handling)
