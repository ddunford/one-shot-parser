# ADR-0004: Structured CV Data Schema

**Status**: Accepted
**Date**: 2025-10-16

## Context

LLMs return unstructured text. We need a standardized JSON schema for parsed CV data that works across all LLM providers and is consumable by clients.

## Decision

Use fixed JSON schema with these sections:
- personalInfo (name, email, phone, address, social links)
- summary (optional career summary)
- skills (categorized: technical, soft, languages, certifications)
- education[] (institution, degree, field, dates, GPA)
- experience[] (company, title, dates, responsibilities, achievements)
- recommendations (job titles, reasoning)

**Validation**: JSON Schema + Ajv validator

## Consequences

✅ **Consistency**: Same structure regardless of LLM provider
✅ **Type Safety**: TypeScript interfaces generated from schema
✅ **Validation**: Automatic validation prevents malformed data
✅ **Documentation**: Self-documenting via schema
❌ **Rigidity**: Hard to add fields without schema change
❌ **LLM Constraints**: Must prompt LLM to match exact schema

## References

- FUNCTIONAL.md § Data Schema Definition
- src/backend/types/cv-data.types.ts

**Related ADRs**: ADR-0001 (Adapter Pattern), ADR-0009 (Prompt Engineering)
