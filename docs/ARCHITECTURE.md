# Architecture Specification - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## System Architecture

### Overview

CV Parser is a stateless, full-stack web application built with a React frontend and Express backend. The system leverages the Adapter Pattern to support multiple LLM providers (Ollama, OpenAI, AWS Bedrock) for intelligent CV parsing. The architecture prioritizes simplicity, privacy, and developer experience with hot reload support throughout.

### Architecture Pattern

**Pattern**: Three-Tier Web Application (Stateless)

```
┌─────────────────────────────────────────────────────────┐
│                     Client Tier                         │
│  React SPA (Vite) - User Interface & Interactions      │
└─────────────────────────────────────────────────────────┘
                            │
                         HTTPS
                            │
┌─────────────────────────────────────────────────────────┐
│                   Reverse Proxy                         │
│         Traefik (External) - cvparse.demosrv.uk         │
└─────────────────────────────────────────────────────────┘
                            │
                         HTTP
                            │
┌─────────────────────────────────────────────────────────┐
│                  Application Tier                       │
│     Express API - Business Logic & Orchestration        │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Upload       │  │ Document     │  │ LLM          │  │
│  │ Handler      │  │ Extractor    │  │ Adapter      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
                            │
                         HTTP
                            │
┌─────────────────────────────────────────────────────────┐
│                   External Services                     │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Ollama       │  │ OpenAI       │  │ AWS Bedrock  │  │
│  │ inference.   │  │ API          │  │ API          │  │
│  │ lan:11434    │  │              │  │              │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Key Characteristics**:
- **Stateless**: No database, no session storage
- **Ephemeral**: Files and data exist only during request lifecycle
- **Scalable**: Horizontal scaling possible (stateless architecture)
- **Privacy-First**: No data retention, no logging of PII

---

## Component Architecture

### Frontend Components

#### 1. Application Shell

**Purpose**: Root component managing routing and global state

**Responsibilities**:
- Application layout
- Provider configuration (Ollama/OpenAI/Bedrock)
- Error boundary for crash recovery
- Global loading states

**Key Files**:
- `src/frontend/App.tsx` - Root component
- `src/frontend/contexts/ProviderContext.tsx` - LLM provider state
- `src/frontend/lib/api.ts` - API client wrapper

**State Management**: React Context API (no Redux needed)

---

#### 2. File Upload Component

**Purpose**: Handle CV file selection and upload

**Responsibilities**:
- Drag-and-drop file handling
- File validation (type, size)
- Upload progress tracking
- Trigger parsing request

**Props**:
```typescript
interface FileUploadProps {
  onFileSelected: (file: File) => void;
  onUploadComplete: (fileId: string) => void;
  acceptedTypes: string[];  // ['.pdf', '.docx']
  maxSize: number;          // 5MB in bytes
  disabled: boolean;
}
```

**Key Files**:
- `src/frontend/components/FileUpload.tsx`
- `src/frontend/components/FilePreview.tsx`
- `src/frontend/hooks/useFileUpload.ts`

---

#### 3. Provider Selector Component

**Purpose**: Allow users to choose LLM provider

**Responsibilities**:
- Display available providers
- Show provider status (online/offline)
- Handle provider selection
- Display provider-specific hints

**Props**:
```typescript
interface ProviderSelectorProps {
  providers: Provider[];
  selected: string;
  onChange: (providerId: string) => void;
  disabled: boolean;
}

interface Provider {
  id: string;                    // 'ollama', 'openai', 'bedrock'
  name: string;                  // 'Ollama (Local)'
  status: 'online' | 'offline' | 'unconfigured';
  description: string;
  estimatedTime: string;         // '20-60 seconds'
}
```

**Key Files**:
- `src/frontend/components/ProviderSelector.tsx`
- `src/frontend/hooks/useProviderStatus.ts`

---

#### 4. Processing Indicator Component

**Purpose**: Show progress during CV parsing

**Responsibilities**:
- Display current processing stage
- Show estimated time remaining
- Provide cancel option (if supported)
- Animate loading states

**Props**:
```typescript
interface ProcessingIndicatorProps {
  stage: 'uploading' | 'extracting' | 'analyzing' | 'complete';
  provider: string;
  progress: number;              // 0-100
  elapsedTime: number;           // seconds
  estimatedTotal: number;        // seconds
}
```

**Key Files**:
- `src/frontend/components/ProcessingIndicator.tsx`
- `src/frontend/components/ProgressBar.tsx`

---

#### 5. Results Display Component

**Purpose**: Present parsed CV data in structured format

**Responsibilities**:
- Render personal information
- Display categorized skills
- Show education timeline
- Show experience timeline
- Handle missing data gracefully

**Props**:
```typescript
interface ResultsDisplayProps {
  cvData: CVData;                // Parsed CV data
  onExport: (format: string) => void;
  onReparse: () => void;
}

interface CVData {
  personalInfo: PersonalInfo;
  summary?: string;
  skills: Skills;
  education: Education[];
  experience: Experience[];
  recommendations?: Recommendations;
}
```

**Key Files**:
- `src/frontend/components/ResultsDisplay.tsx`
- `src/frontend/components/PersonalInfoCard.tsx`
- `src/frontend/components/SkillsSection.tsx`
- `src/frontend/components/EducationTimeline.tsx`
- `src/frontend/components/ExperienceTimeline.tsx`

---

#### 6. JSON Viewer Component

**Purpose**: Display and allow copying of raw JSON output

**Responsibilities**:
- Pretty-print JSON
- Syntax highlighting
- Copy to clipboard
- Expand/collapse functionality

**Props**:
```typescript
interface JSONViewerProps {
  data: object;
  onCopy: () => void;
  isExpanded: boolean;
  onToggle: () => void;
}
```

**Key Files**:
- `src/frontend/components/JSONViewer.tsx`
- `src/frontend/hooks/useClipboard.ts`

---

### Backend Components

#### 1. Express Application

**Purpose**: HTTP server and request routing

**Responsibilities**:
- Route registration
- Middleware configuration
- CORS handling
- Error handling
- Health checks

**Key Files**:
- `src/backend/index.ts` - Application entry point
- `src/backend/app.ts` - Express app configuration
- `src/backend/config/server.ts` - Server configuration

**Middleware Stack**:
```typescript
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors(corsOptions));
app.use(helmet());
app.use(rateLimit(rateLimitOptions));
app.use(requestLogger);
```

---

#### 2. Upload Controller

**Purpose**: Handle file upload requests

**Responsibilities**:
- Receive multipart form data
- Validate file type and size
- Temporarily store file
- Pass to document extractor
- Clean up after processing

**Endpoint**: `POST /api/parse`

**Request**:
```typescript
// multipart/form-data
{
  file: File;              // PDF or DOCX
  provider?: string;       // 'ollama' | 'openai' | 'bedrock'
}
```

**Response**:
```typescript
{
  success: boolean;
  data?: CVData;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  meta: {
    processingTime: number;
    provider: string;
  };
}
```

**Key Files**:
- `src/backend/controllers/parseController.ts`
- `src/backend/middleware/fileUpload.ts` (Multer config)
- `src/backend/validators/fileValidator.ts`

---

#### 3. Document Extractor Service

**Purpose**: Extract text from PDF and DOCX files

**Responsibilities**:
- PDF text extraction (using pdf-parse)
- DOCX text extraction (using mammoth)
- Handle extraction errors
- Preserve document structure
- Clean and normalize text

**Interface**:
```typescript
interface IDocumentExtractor {
  extract(file: Buffer, mimeType: string): Promise<ExtractedText>;
}

interface ExtractedText {
  text: string;
  pageCount?: number;
  wordCount: number;
  metadata: {
    title?: string;
    author?: string;
    createdDate?: Date;
  };
}
```

**Implementation**:
```typescript
class PDFExtractor implements IDocumentExtractor {
  async extract(buffer: Buffer): Promise<ExtractedText> {
    const data = await pdf(buffer);
    return {
      text: data.text,
      pageCount: data.numpages,
      wordCount: data.text.split(/\s+/).length,
      metadata: data.info
    };
  }
}

class DOCXExtractor implements IDocumentExtractor {
  async extract(buffer: Buffer): Promise<ExtractedText> {
    const result = await mammoth.extractRawText({ buffer });
    return {
      text: result.value,
      wordCount: result.value.split(/\s+/).length,
      metadata: {}
    };
  }
}
```

**Key Files**:
- `src/backend/services/extractors/DocumentExtractor.ts` (interface)
- `src/backend/services/extractors/PDFExtractor.ts`
- `src/backend/services/extractors/DOCXExtractor.ts`
- `src/backend/services/extractors/ExtractorFactory.ts`

---

#### 4. LLM Adapter (Strategy Pattern)

**Purpose**: Abstract LLM provider differences behind common interface

**Responsibilities**:
- Define common interface for all providers
- Implement provider-specific logic
- Handle authentication
- Format requests and parse responses
- Error handling and retries

**Interface**:
```typescript
interface ILLMAdapter {
  parseCV(text: string, options?: ParseOptions): Promise<CVData>;
  getProviderInfo(): ProviderInfo;
  checkHealth(): Promise<boolean>;
}

interface ParseOptions {
  timeout?: number;           // milliseconds
  temperature?: number;       // 0-1
  maxTokens?: number;
}

interface ProviderInfo {
  id: string;
  name: string;
  status: 'online' | 'offline';
  latency?: number;           // ms
}
```

**Implementations**:

##### Ollama Adapter
```typescript
class OllamaAdapter implements ILLMAdapter {
  private baseUrl: string;
  private model: string;

  constructor(config: { baseUrl: string; model: string }) {
    this.baseUrl = config.baseUrl;  // http://inference.lan:11434
    this.model = config.model;      // e.g., 'llama3.1'
  }

  async parseCV(text: string, options?: ParseOptions): Promise<CVData> {
    const prompt = this.buildPrompt(text);

    const response = await fetch(`${this.baseUrl}/api/chat`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: this.model,
        messages: [
          { role: 'system', content: SYSTEM_PROMPT },
          { role: 'user', content: prompt }
        ],
        format: 'json',
        stream: false,
        options: {
          temperature: options?.temperature || 0.1
        }
      }),
      signal: AbortSignal.timeout(options?.timeout || 60000)
    });

    const result = await response.json();
    return this.parseResponse(result.message.content);
  }
}
```

##### OpenAI Adapter
```typescript
class OpenAIAdapter implements ILLMAdapter {
  private apiKey: string;
  private model: string;

  constructor(config: { apiKey: string; model: string }) {
    this.apiKey = config.apiKey;
    this.model = config.model;      // 'gpt-4' or 'gpt-3.5-turbo'
  }

  async parseCV(text: string, options?: ParseOptions): Promise<CVData> {
    const openai = new OpenAI({ apiKey: this.apiKey });

    const completion = await openai.chat.completions.create({
      model: this.model,
      messages: [
        { role: 'system', content: SYSTEM_PROMPT },
        { role: 'user', content: this.buildPrompt(text) }
      ],
      response_format: { type: 'json_object' },
      temperature: options?.temperature || 0.1,
      max_tokens: options?.maxTokens || 2000,
      timeout: options?.timeout || 60000
    });

    const content = completion.choices[0]?.message?.content;
    return this.parseResponse(content);
  }
}
```

##### Bedrock Adapter
```typescript
class BedrockAdapter implements ILLMAdapter {
  private client: BedrockRuntimeClient;
  private modelId: string;

  constructor(config: { region: string; modelId: string }) {
    this.client = new BedrockRuntimeClient({ region: config.region });
    this.modelId = config.modelId;  // 'anthropic.claude-v2'
  }

  async parseCV(text: string, options?: ParseOptions): Promise<CVData> {
    const command = new InvokeModelCommand({
      modelId: this.modelId,
      body: JSON.stringify({
        prompt: this.buildPrompt(text),
        max_tokens_to_sample: options?.maxTokens || 2000,
        temperature: options?.temperature || 0.1
      })
    });

    const response = await this.client.send(command);
    const result = JSON.parse(new TextDecoder().decode(response.body));
    return this.parseResponse(result.completion);
  }
}
```

**Key Files**:
- `src/backend/services/llm/ILLMAdapter.ts` (interface)
- `src/backend/services/llm/OllamaAdapter.ts`
- `src/backend/services/llm/OpenAIAdapter.ts`
- `src/backend/services/llm/BedrockAdapter.ts`
- `src/backend/services/llm/AdapterFactory.ts`
- `src/backend/services/llm/prompts.ts` (shared prompts)

---

#### 5. Parse Service (Orchestrator)

**Purpose**: Orchestrate the complete parsing workflow

**Responsibilities**:
- Receive file and provider selection
- Extract text from document
- Select and invoke LLM adapter
- Validate response schema
- Handle errors and timeouts
- Return structured result

**Interface**:
```typescript
interface IParseService {
  parseCV(file: File, provider: string): Promise<ParseResult>;
}

interface ParseResult {
  success: boolean;
  data?: CVData;
  error?: Error;
  meta: {
    processingTime: number;
    provider: string;
    stages: {
      extraction: number;
      llmProcessing: number;
      validation: number;
    };
  };
}
```

**Implementation**:
```typescript
class ParseService implements IParseService {
  private extractorFactory: ExtractorFactory;
  private adapterFactory: AdapterFactory;
  private validator: CVDataValidator;

  async parseCV(file: Buffer, mimeType: string, provider: string): Promise<ParseResult> {
    const startTime = Date.now();
    const stages = { extraction: 0, llmProcessing: 0, validation: 0 };

    try {
      // Stage 1: Text Extraction
      const extractStart = Date.now();
      const extractor = this.extractorFactory.getExtractor(mimeType);
      const extracted = await extractor.extract(file);
      stages.extraction = Date.now() - extractStart;

      // Stage 2: LLM Processing
      const llmStart = Date.now();
      const adapter = this.adapterFactory.getAdapter(provider);
      const cvData = await adapter.parseCV(extracted.text);
      stages.llmProcessing = Date.now() - llmStart;

      // Stage 3: Validation
      const validationStart = Date.now();
      const validatedData = await this.validator.validate(cvData);
      stages.validation = Date.now() - validationStart;

      return {
        success: true,
        data: validatedData,
        meta: {
          processingTime: Date.now() - startTime,
          provider,
          stages
        }
      };
    } catch (error) {
      return {
        success: false,
        error: error as Error,
        meta: {
          processingTime: Date.now() - startTime,
          provider,
          stages
        }
      };
    }
  }
}
```

**Key Files**:
- `src/backend/services/ParseService.ts`
- `src/backend/validators/CVDataValidator.ts`

---

## Data Flow

### Primary Flow: CV Upload to Parsed Results

```
┌──────────────────────────────────────────────────────────────────────┐
│ 1. User Uploads File                                                  │
│    Frontend: FileUpload component                                     │
│    - File selected via drag-drop or click                             │
│    - Client-side validation (type, size)                              │
│    - File preview displayed                                           │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 2. POST /api/parse Request                                            │
│    Frontend: API client                                               │
│    - multipart/form-data with file and provider                       │
│    - Headers: Content-Type, Accept                                    │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 3. File Upload Middleware                                             │
│    Backend: Multer middleware                                         │
│    - Receive file buffer                                              │
│    - Server-side validation (size, type, content)                     │
│    - Store in memory (temporary)                                      │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 4. Parse Controller                                                   │
│    Backend: parseController.parseCV()                                 │
│    - Extract request parameters                                       │
│    - Invoke ParseService                                              │
│    - Handle response or error                                         │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 5. Document Extraction                                                │
│    Backend: DocumentExtractor                                         │
│    - Determine file type (PDF/DOCX)                                   │
│    - Extract text with appropriate library                            │
│    - Clean and normalize text                                         │
│    - Return extracted text (3-5 seconds)                              │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 6. LLM Adapter Selection                                              │
│    Backend: AdapterFactory                                            │
│    - Select adapter based on provider parameter                       │
│    - Initialize with configuration                                    │
│    - Check provider health/availability                               │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 7. LLM Request                                                        │
│    Backend: LLMAdapter (Ollama/OpenAI/Bedrock)                        │
│    - Build structured prompt with CV text                             │
│    - Send to LLM provider                                             │
│    - Wait for response (10-60 seconds)                                │
│    - Handle timeout (60s max)                                         │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 8. Response Parsing                                                   │
│    Backend: LLMAdapter.parseResponse()                                │
│    - Parse JSON from LLM                                              │
│    - Map to CVData schema                                             │
│    - Handle missing/invalid fields                                    │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 9. Data Validation                                                    │
│    Backend: CVDataValidator                                           │
│    - Validate against JSON schema                                     │
│    - Type checking                                                    │
│    - Format validation (emails, dates)                                │
│    - Sanitization                                                     │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 10. HTTP Response                                                     │
│    Backend: parseController response                                  │
│    - Status: 200 OK                                                   │
│    - Body: { success: true, data: CVData, meta: {...} }              │
│    - Clean up temporary files                                         │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 11. Display Results                                                   │
│    Frontend: ResultsDisplay component                                 │
│    - Render PersonalInfo card                                         │
│    - Render Skills section                                            │
│    - Render Education timeline                                        │
│    - Render Experience timeline                                       │
│    - Render Job Recommendations (if available)                        │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│ 12. JSON Export (Optional)                                            │
│    Frontend: JSONViewer component                                     │
│    - User clicks "View JSON"                                          │
│    - JSON displayed with syntax highlighting                          │
│    - User clicks "Copy to Clipboard"                                  │
│    - JSON copied, success confirmation shown                          │
└──────────────────────────────────────────────────────────────────────┘
```

**Total Time**: 15-70 seconds (depends on LLM provider and CV complexity)

---

## Technology Stack Justification

### Frontend: React + Vite

**Why React?**
- ✅ Component-based architecture (reusable UI components)
- ✅ Mature ecosystem with excellent libraries
- ✅ Strong TypeScript support
- ✅ Great developer experience
- ✅ Suitable for interactive SPA

**Why Vite?**
- ✅ Extremely fast hot module reload (HMR)
- ✅ Optimized for development speed
- ✅ Out-of-the-box TypeScript support
- ✅ Modern build tool (ESM-based)
- ✅ Smaller bundle size than Create React App

**Alternatives Considered**:
- ❌ Next.js: Overkill (no SSR needed, stateless app)
- ❌ Vue: Team familiarity with React
- ❌ Svelte: Smaller ecosystem, less mature tooling

---

### Backend: Express (Node.js)

**Why Express?**
- ✅ Lightweight and flexible
- ✅ Perfect for stateless API
- ✅ Mature ecosystem (Multer, cors, helmet, etc.)
- ✅ Easy hot reload with nodemon
- ✅ Excellent async/await support
- ✅ Fast time-to-market

**Why Node.js?**
- ✅ Same language (TypeScript) as frontend
- ✅ Great for I/O-heavy operations (file uploads, API calls)
- ✅ Extensive LLM client libraries (Ollama, OpenAI, Bedrock)
- ✅ Fast development cycle

**Alternatives Considered**:
- ❌ NestJS: Over-engineered for simple API
- ❌ Fastify: Learning curve, not necessary for this scale
- ❌ Python/Django: Different language, slower iteration
- ❌ Go: Great performance but overkill, less library support

---

### Document Processing Libraries

**PDF: pdf-parse**
- ✅ Pure JavaScript (no dependencies)
- ✅ Handles most PDF formats
- ✅ Extracts text with layout preservation
- ✅ Lightweight

**DOCX: mammoth**
- ✅ Well-maintained library
- ✅ Clean text extraction
- ✅ Handles complex Word documents
- ✅ Simple API

**Alternatives Considered**:
- ❌ pdf.js: Browser-focused, heavyweight
- ❌ docx4js: Less mature
- ❌ Apache Tika: Java dependency, complex setup

---

### LLM Integration

**Ollama Client: ollama-js**
- ✅ Official JavaScript library
- ✅ Simple chat API
- ✅ Streaming support (future)
- ✅ JSON response format

**OpenAI Client: openai-node**
- ✅ Official TypeScript library
- ✅ Structured outputs with Zod
- ✅ Type-safe
- ✅ Comprehensive error handling

**AWS Bedrock: @aws-sdk/client-bedrock-runtime**
- ✅ Official AWS SDK
- ✅ IAM authentication
- ✅ Multiple model support (Claude, Llama, etc.)
- ✅ Enterprise-ready

---

## Design Patterns

### 1. Adapter Pattern (LLM Integration)

**Problem**: Multiple LLM providers with different APIs

**Solution**: Common interface (`ILLMAdapter`) with provider-specific implementations

**Benefits**:
- Provider switching without code changes
- Easy to add new providers
- Testable with mock adapters
- Clean separation of concerns

---

### 2. Factory Pattern (Extractor & Adapter Selection)

**Problem**: Need to instantiate different extractors/adapters based on runtime conditions

**Solution**: Factory classes that create appropriate instances

**Benefits**:
- Centralized object creation logic
- Easy to extend with new types
- Reduced coupling

---

### 3. Strategy Pattern (Document Extraction)

**Problem**: Different extraction logic for PDF vs DOCX

**Solution**: Common interface (`IDocumentExtractor`) with format-specific strategies

**Benefits**:
- Interchangeable extraction algorithms
- Open-closed principle (add formats without modifying existing code)
- Testable in isolation

---

### 4. Repository Pattern (Future: If Database Added)

**Problem**: Data access abstraction (not applicable in stateless MVP)

**Solution**: N/A (no database in MVP)

**Future Consideration**: If user accounts or parsing history added

---

## Scalability Strategy

### Horizontal Scaling

**Current**: Single instance, suitable for <100 concurrent users

**Future Scaling**:
1. **Load Balancer**: Add load balancer in front of multiple backend instances
2. **Traefik**: Already supports load balancing to multiple containers
3. **Stateless Architecture**: Enables trivial horizontal scaling
4. **Docker Compose Scale**: `docker compose up --scale backend=3`

**No Changes Needed**: Application is already stateless and scale-ready

---

### Performance Optimization

**Current Bottlenecks**:
1. LLM processing time (10-60 seconds) - Provider-dependent
2. Large file uploads (5MB max) - Network-dependent
3. PDF extraction for complex documents - CPU-bound

**Future Optimizations**:
1. **Response Streaming**: Stream LLM responses for faster perceived performance
2. **Caching**: Cache parsed results by file hash (requires storage)
3. **Queue System**: Offload long-running LLM requests to queue (Celery, Bull)
4. **CDN**: Cache static frontend assets
5. **Compression**: gzip/Brotli compression for API responses

---

## Security Architecture

### Threat Model

**Assets to Protect**:
1. CV documents (contain PII)
2. API keys (LLM providers)
3. System availability (prevent DOS)

**Threat Actors**:
1. Malicious users uploading harmful files
2. Automated bots spamming upload endpoint
3. Attackers attempting to steal API keys
4. Attackers attempting LLM prompt injection

**Mitigations**: See SECURITY.md for detailed analysis

---

## Deployment Architecture

### Development Environment

```yaml
services:
  backend:
    build:
      context: ./src/backend
      target: development
    volumes:
      - ./src/backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - OLLAMA_URL=http://inference.lan:11434
    ports:
      - "3001:3001"
    networks:
      - traefik_demosrv
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cvparse-api.rule=Host(`cvparse.demosrv.uk`) && PathPrefix(`/api`)"
      - "traefik.http.services.cvparse-api.loadbalancer.server.port=3001"

  frontend:
    build:
      context: ./src/frontend
      target: development
    volumes:
      - ./src/frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - VITE_API_URL=https://cvparse.demosrv.uk/api
    networks:
      - traefik_demosrv
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cvparse.rule=Host(`cvparse.demosrv.uk`)"
      - "traefik.http.routers.cvparse.entrypoints=websecure"
      - "traefik.http.routers.cvparse.tls.certresolver=letsencrypt"
      - "traefik.http.services.cvparse.loadbalancer.server.port=3000"

networks:
  traefik_demosrv:
    external: true
```

**Key Features**:
- Hot reload for both services
- Traefik routing to both frontend and backend
- External Traefik network (already running)
- HTTPS via Let's Encrypt

---

### Production Environment

**Changes from Development**:
- Use `target: production` in Dockerfile
- No volume mounts (compiled code)
- Optimized builds (minified, tree-shaken)
- Health checks enabled
- Resource limits applied
- Logging to stdout (Docker captures)

---

## Monitoring & Observability

### Logging Strategy

**What to Log**:
- Request/response metadata (no PII)
- Processing times per stage
- Error stack traces
- Provider selection and latency

**What NOT to Log**:
- CV file contents
- Extracted text
- Parsed personal information
- API keys

**Log Format**: Structured JSON logs

**Example Log**:
```json
{
  "timestamp": "2025-10-16T14:30:00Z",
  "level": "info",
  "service": "cv-parser-backend",
  "event": "cv_parsed",
  "meta": {
    "processingTime": 32000,
    "provider": "ollama",
    "fileSize": 2048576,
    "fileType": "application/pdf",
    "success": true,
    "stages": {
      "extraction": 3200,
      "llmProcessing": 28000,
      "validation": 800
    }
  }
}
```

---

### Health Checks

**Backend Health Endpoint**: `GET /api/health`

**Response**:
```json
{
  "status": "healthy",
  "timestamp": "2025-10-16T14:30:00Z",
  "uptime": 86400,
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

**Traefik Health Check**:
```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "--quiet", "http://localhost:3001/api/health"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 40s
```

---

### Metrics (Future)

**Proposed Metrics**:
- Requests per minute
- Average processing time by provider
- Error rate by error type
- File type distribution
- Provider usage distribution

**Tools**: Prometheus + Grafana (future addition)

---

## Appendix

### Directory Structure

```
.
├── .autoflow/                    # Auto-flow documentation
│   ├── docs/                     # Design docs (this file)
│   ├── adrs/                     # Architecture Decision Records
│   ├── SPRINTS.yml               # Sprint planning
│   └── BUILD_SPEC.md             # Project configuration
├── src/
│   ├── backend/
│   │   ├── controllers/
│   │   │   └── parseController.ts
│   │   ├── services/
│   │   │   ├── ParseService.ts
│   │   │   ├── extractors/
│   │   │   │   ├── IDocumentExtractor.ts
│   │   │   │   ├── PDFExtractor.ts
│   │   │   │   ├── DOCXExtractor.ts
│   │   │   │   └── ExtractorFactory.ts
│   │   │   └── llm/
│   │   │       ├── ILLMAdapter.ts
│   │   │       ├── OllamaAdapter.ts
│   │   │       ├── OpenAIAdapter.ts
│   │   │       ├── BedrockAdapter.ts
│   │   │       ├── AdapterFactory.ts
│   │   │       └── prompts.ts
│   │   ├── middleware/
│   │   │   ├── fileUpload.ts
│   │   │   ├── errorHandler.ts
│   │   │   └── rateLimiter.ts
│   │   ├── validators/
│   │   │   ├── CVDataValidator.ts
│   │   │   └── fileValidator.ts
│   │   ├── types/
│   │   │   └── cv-data.types.ts
│   │   ├── config/
│   │   │   └── server.ts
│   │   ├── app.ts
│   │   ├── index.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── Dockerfile
│   └── frontend/
│       ├── components/
│       │   ├── FileUpload.tsx
│       │   ├── ProviderSelector.tsx
│       │   ├── ProcessingIndicator.tsx
│       │   ├── ResultsDisplay.tsx
│       │   ├── PersonalInfoCard.tsx
│       │   ├── SkillsSection.tsx
│       │   ├── EducationTimeline.tsx
│       │   ├── ExperienceTimeline.tsx
│       │   └── JSONViewer.tsx
│       ├── hooks/
│       │   ├── useFileUpload.ts
│       │   ├── useProviderStatus.ts
│       │   └── useClipboard.ts
│       ├── contexts/
│       │   └── ProviderContext.tsx
│       ├── lib/
│       │   └── api.ts
│       ├── types/
│       │   └── cv-data.types.ts
│       ├── App.tsx
│       ├── main.tsx
│       ├── package.json
│       ├── tsconfig.json
│       ├── vite.config.ts
│       └── Dockerfile
├── docker-compose.yml
├── CLAUDE.md
├── tests.sh
└── README.md
```

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 1
**Owner**: Engineering Team
