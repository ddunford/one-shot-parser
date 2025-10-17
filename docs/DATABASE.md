# Database Specification - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## Overview

**Database Type**: None (Stateless Architecture)

The CV Parser application operates entirely without a database. All data processing happens in-memory during the request lifecycle and is discarded after the response is sent.

---

## Rationale for No Database

### Design Decision

**Decision**: Stateless, no-database architecture

**Reasons**:
1. **Privacy**: CV documents contain PII - not storing them protects user privacy
2. **Simplicity**: No database migrations, backups, or maintenance
3. **Scalability**: Stateless architecture enables trivial horizontal scaling
4. **Cost**: No database hosting costs
5. **Compliance**: Easier to comply with data protection regulations (GDPR, CCPA)

**Trade-offs**:
- ❌ No user accounts or authentication
- ❌ No parsing history
- ❌ No analytics on individual users
- ❌ No ability to retrieve past results
- ✅ Zero data retention risks
- ✅ Horizontal scaling without shared state
- ✅ Fast deployment (no schema management)

---

## Data Lifecycle

### Request Lifecycle

```
1. File Upload
   ├─ File stored in memory (Buffer)
   └─ Lifetime: Request duration only

2. Text Extraction
   ├─ Extracted text in memory (string)
   └─ Lifetime: Request duration only

3. LLM Processing
   ├─ Prompt sent to external LLM
   └─ Response stored in memory

4. Response
   ├─ CVData returned to client
   └─ Server memory cleared

5. Cleanup
   └─ All data discarded (garbage collected)
```

**Total Data Lifetime**: ~10-70 seconds (from upload to response)

**Persistence**: None

---

## Future Database Considerations

If the application evolves to require persistence, consider these options:

### Option 1: Redis (Caching Layer)

**Use Case**: Temporarily cache parsing results by file hash

**Schema**:
```
Key: cv:{fileHash}
Value: {
  cvData: CVData,
  timestamp: ISO8601,
  provider: string
}
TTL: 1 hour
```

**Benefits**:
- Fast lookup for repeat parses
- No permanent storage
- Auto-expiry
- Horizontal scaling support

---

### Option 2: PostgreSQL (Full Persistence)

**Use Case**: User accounts, parsing history, analytics

**Proposed Tables**:

#### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  last_login TIMESTAMP
);
```

#### parsing_history
```sql
CREATE TABLE parsing_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  file_hash VARCHAR(64) UNIQUE,  -- SHA256 of file
  file_name VARCHAR(255),
  file_size INTEGER,
  provider VARCHAR(50),
  processing_time INTEGER,       -- milliseconds
  success BOOLEAN,
  parsed_at TIMESTAMP DEFAULT NOW(),
  cv_data JSONB                  -- Parsed CV data
);

CREATE INDEX idx_parsing_history_user_id ON parsing_history(user_id);
CREATE INDEX idx_parsing_history_parsed_at ON parsing_history(parsed_at DESC);
```

#### api_usage
```sql
CREATE TABLE api_usage (
  id SERIAL PRIMARY KEY,
  ip_address INET,
  endpoint VARCHAR(255),
  method VARCHAR(10),
  status_code INTEGER,
  processing_time INTEGER,
  timestamp TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_api_usage_ip_timestamp ON api_usage(ip_address, timestamp);
```

---

### Option 3: MongoDB (Document Store)

**Use Case**: Flexible schema for varying CV structures

**Collections**:

#### cvs
```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  fileHash: string,
  fileName: string,
  provider: string,
  parsedAt: Date,
  cvData: {
    personalInfo: {},
    skills: {},
    education: [],
    experience: []
  },
  metadata: {
    processingTime: number,
    fileSize: number,
    success: boolean
  }
}
```

**Indexes**:
```javascript
db.cvs.createIndex({ userId: 1, parsedAt: -1 });
db.cvs.createIndex({ fileHash: 1 }, { unique: true });
```

---

## Data Protection (If Database Added)

**Required Measures**:

1. **Encryption at Rest**: All CV data encrypted in database
2. **Encryption in Transit**: TLS for all database connections
3. **Access Control**: Principle of least privilege
4. **Data Retention**: Auto-delete after 30 days
5. **GDPR Compliance**: Right to erasure, data portability
6. **Audit Logging**: Track all data access
7. **Backups**: Encrypted backups with limited retention

---

## Current State

**Storage**: In-memory only (Node.js process memory)

**Persistence**: None

**Backup**: Not applicable

**Migration Strategy**: Not applicable

**Data Recovery**: Not applicable (data is ephemeral)

---

**Document Status**: ✅ Complete (N/A for stateless architecture)
**Next Review Date**: If database requirement emerges
**Owner**: Engineering Team
