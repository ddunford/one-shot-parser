# Product Specification - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## Executive Summary

CV Parser is a streamlined web application that enables users to upload CV/resume documents and receive structured, AI-parsed data. The application leverages Large Language Models (LLMs) to extract personal information, skills, education, work experience, and generate job title recommendations. With support for multiple LLM providers (Ollama, OpenAI, AWS Bedrock), the tool offers flexibility in AI processing while maintaining a simple, no-authentication user experience.

**Key Value Proposition**: Transform unstructured CV documents into structured, machine-readable data in seconds, with the flexibility to choose from multiple AI providers.

---

## Target Users & Personas

### Primary Persona: HR Recruiter (Sarah)

**Demographics**:
- Age: 28-45
- Role: HR Manager / Recruiter
- Tech-savviness: Moderate

**Goals**:
- Quickly extract structured data from candidate CVs
- Standardize CV information for applicant tracking systems
- Reduce manual data entry time

**Pain Points**:
- CVs come in various formats and structures
- Manual extraction is time-consuming and error-prone
- Need quick turnaround for high-volume hiring

**How This Tool Helps**:
- Instant structured data extraction
- Consistent output format (JSON)
- No login required - frictionless usage

### Secondary Persona: Job Seeker (Alex)

**Demographics**:
- Age: 22-35
- Role: Software Developer / Professional
- Tech-savviness: High

**Goals**:
- Validate CV formatting and completeness
- See how AI interprets their resume
- Get job title recommendations

**Pain Points**:
- Unsure if ATS systems can parse their CV correctly
- Want to optimize CV for AI parsing
- Need career guidance

**How This Tool Helps**:
- Instant feedback on CV parseability
- Structured view of extracted data
- AI-generated job recommendations

### Tertiary Persona: Hiring Platform Developer (Marcus)

**Demographics**:
- Age: 25-40
- Role: Backend Developer / Integration Engineer
- Tech-savviness: Expert

**Goals**:
- Integrate CV parsing into existing systems
- Standardized API for CV data extraction
- Flexible LLM provider selection

**Pain Points**:
- Building CV parsing from scratch is complex
- Need to support multiple LLM providers
- Want consistent JSON output

**How This Tool Helps**:
- Simple REST API
- Standardized JSON schema
- Multi-provider support

---

## Product Features

### MVP Features (Must Have)

#### 1. File Upload

**Description**: Drag-and-drop or click-to-upload interface supporting PDF and DOCX formats.

**User Story**: As a recruiter, I want to upload a CV file quickly so that I can get parsed results without friction.

**Acceptance Criteria**:
- Support PDF and DOCX file formats
- File size limit: 5MB
- Visual feedback during upload (progress bar)
- Clear error messages for invalid files
- Drag-and-drop support

#### 2. Document Text Extraction

**Description**: Extract raw text content from uploaded CV documents.

**User Story**: As a system, I need to extract text from various document formats so that I can send it to the LLM for parsing.

**Acceptance Criteria**:
- PDF text extraction with layout preservation
- DOCX text extraction maintaining structure
- Handle multi-column layouts
- Extract text from images (OCR if possible)
- Processing time: <5 seconds for typical CV

#### 3. LLM-Powered CV Parsing

**Description**: Use AI to intelligently parse CV content into structured fields.

**User Story**: As a recruiter, I want the system to automatically identify and extract key CV components so that I don't have to read through entire documents manually.

**Acceptance Criteria**:
- Extract personal information (name, email, phone, address)
- Categorize skills by type (technical, soft, languages)
- Parse education history with dates and institutions
- Parse work experience with dates, companies, and roles
- Processing time: 10-60 seconds depending on LLM provider
- Graceful handling of missing or ambiguous information

#### 4. Ollama Integration (Default Provider)

**Description**: Connect to local Ollama instance for free, local LLM inference.

**User Story**: As a developer, I want to use my local Ollama instance so that I can process CVs without incurring API costs.

**Acceptance Criteria**:
- Connect to http://inference.lan:11434
- Support any Ollama-compatible model
- Handle connection errors gracefully
- Display provider status in UI
- Configurable timeout (60 seconds default)

#### 5. Structured Results Display

**Description**: Present parsed CV data in a clean, readable format with clear sections.

**User Story**: As a recruiter, I want to see the parsed CV data organized by category so that I can quickly assess a candidate's qualifications.

**Acceptance Criteria**:
- Personal Information section
- Skills section (categorized: technical, soft, languages, certifications)
- Education History section (chronological)
- Work Experience section (reverse chronological)
- Clean, scannable layout
- Responsive design for mobile/tablet/desktop

#### 6. JSON Export

**Description**: Provide raw JSON output with copy-to-clipboard functionality.

**User Story**: As a developer, I want to copy the parsed CV data as JSON so that I can integrate it into my own systems.

**Acceptance Criteria**:
- Expandable JSON viewer section
- Pretty-printed JSON with syntax highlighting
- One-click copy to clipboard
- Copy success confirmation
- Valid JSON schema adherence

---

### Post-MVP Features (Should Have)

#### 7. OpenAI Integration

**Description**: Support OpenAI GPT models for higher-quality parsing.

**User Story**: As a user, I want to use OpenAI's models when I need the highest quality results for important CVs.

**Acceptance Criteria**:
- Provider selection dropdown in UI
- API key configuration (environment variable)
- Support for GPT-4 and GPT-3.5-turbo
- Cost estimation display (optional)
- Same JSON schema output as Ollama

#### 8. AWS Bedrock Integration

**Description**: Support AWS Bedrock for enterprise users with AWS infrastructure.

**User Story**: As an enterprise user, I want to use AWS Bedrock so that my CV data stays within our AWS environment.

**Acceptance Criteria**:
- AWS credentials configuration
- Support for Claude and other Bedrock models
- Region selection
- IAM role authentication support
- Enterprise-grade error handling

#### 9. Job Title Recommendations

**Description**: AI-generated suggestions for appropriate job titles based on skills and experience.

**User Story**: As a job seeker, I want to see what job titles match my skills so that I can target my applications better.

**Acceptance Criteria**:
- 3-5 job title recommendations
- Brief reasoning for each recommendation
- Based on skills, experience level, and industry
- Displayed prominently in results
- Confidence score for each recommendation (optional)

#### 10. Advanced Error Handling

**Description**: Comprehensive error handling with user-friendly messages and retry mechanisms.

**User Story**: As a user, I want clear feedback when something goes wrong so that I know how to resolve the issue.

**Acceptance Criteria**:
- Specific error messages (file type, size, LLM timeout, etc.)
- Retry button for transient failures
- Troubleshooting tips in error messages
- Graceful degradation (partial results if parsing incomplete)
- Error logging (backend only, no PII)

---

### Nice-to-Have Features (Could Have)

#### 11. Multiple Provider Comparison

**Description**: Allow users to parse the same CV with different providers and compare results.

**User Story**: As a user, I want to see how different AI models parse my CV so that I can understand which provider works best.

**Acceptance Criteria**:
- "Compare Providers" mode
- Side-by-side results display
- Highlight differences between providers
- Processing time comparison
- Quality metrics (completeness, accuracy)

#### 12. Export to Other Formats

**Description**: Export parsed data to CSV, LinkedIn profile format, or PDF report.

**User Story**: As a recruiter, I want to export CV data in different formats so that I can use it in various tools.

**Acceptance Criteria**:
- CSV export for spreadsheet tools
- LinkedIn profile format
- PDF report with formatted layout
- Email-friendly HTML format
- Batch export for multiple CVs (future)

#### 13. CV Quality Scoring

**Description**: Provide a score and feedback on CV quality and completeness.

**User Story**: As a job seeker, I want to know how good my CV is so that I can improve it before applying.

**Acceptance Criteria**:
- Overall score (0-100)
- Completeness checklist
- Formatting suggestions
- Keyword optimization tips
- ATS-friendliness assessment

#### 14. Anonymous Usage Analytics

**Description**: Track aggregated usage metrics without collecting PII.

**User Story**: As a product owner, I want to understand usage patterns so that I can improve the product.

**Acceptance Criteria**:
- File format distribution
- Average processing time per provider
- Error rate tracking
- No CV content or personal data logged
- Privacy-compliant analytics

---

### Out of Scope (Won't Have)

- **User Authentication**: No login system required
- **Database Storage**: Completely stateless application
- **CV Editing**: No capability to modify uploaded CVs
- **Candidate Tracking**: No applicant tracking system features
- **Team Collaboration**: No sharing or multi-user workflows
- **Historical Records**: No saved parsing history
- **Batch Processing**: Process one CV at a time (for now)

---

## User Journeys

### Journey 1: Basic CV Upload & Parse (Primary Flow)

1. **Landing Page**
   - User visits https://cvparse.demosrv.uk
   - Sees clear hero section explaining the tool
   - Large "Upload CV" button prominently displayed
   - Optional: Sample CV for demo purposes

2. **File Selection**
   - User clicks "Upload CV" or drags file to drop zone
   - File picker opens (filters: .pdf, .docx)
   - User selects CV file from computer
   - File appears in upload preview with name and size

3. **Provider Selection (Optional)**
   - Dropdown shows available LLM providers
   - Default: Ollama (local inference)
   - Provider status indicators (online/offline)
   - User keeps default or selects alternative

4. **Initiate Parsing**
   - User clicks "Parse CV" button
   - Button shows loading state
   - Progress indicator appears
   - Estimated time displayed (10-60 seconds)

5. **Processing Feedback**
   - "Extracting text from document..." (0-5s)
   - "Analyzing with AI..." (10-60s)
   - "Structuring results..." (1-2s)
   - Real-time progress updates

6. **Results Display**
   - Page scrolls to results section
   - Personal Information card displayed first
   - Skills section (categorized and tagged)
   - Education timeline
   - Work Experience timeline (reverse chronological)
   - Job Title Recommendations (if enabled)

7. **JSON Export**
   - Expandable "View JSON" section at bottom
   - Syntax-highlighted JSON display
   - "Copy to Clipboard" button
   - Success confirmation on copy

8. **Next Actions**
   - "Parse Another CV" button to restart
   - "Try Different Provider" to re-parse with alternative
   - Share results link (future feature)

**Success Metrics**:
- Time to first result: <60 seconds
- User satisfaction with parsing accuracy: >80%
- Completion rate: >70%

---

### Journey 2: Error Recovery (Alternative Flow)

1. **Invalid File Upload**
   - User uploads .txt file instead of .pdf/.docx
   - Red error banner: "Unsupported file format. Please upload PDF or DOCX."
   - Upload area highlights in red
   - User selects correct file type

2. **File Too Large**
   - User uploads 8MB file (limit: 5MB)
   - Error: "File size exceeds 5MB limit. Please compress or split document."
   - Suggestion: "Most CVs are 1-2MB. Consider removing images or optimizing PDF."
   - User optimizes and re-uploads

3. **LLM Connection Failure**
   - User initiates parsing
   - After 5 seconds: "Connecting to Ollama... This is taking longer than usual."
   - After 10 seconds: Error banner appears
   - Message: "Cannot connect to Ollama at http://inference.lan:11434. Please ensure Ollama is running."
   - Action buttons: "Retry" or "Try OpenAI Instead"
   - User either fixes Ollama or switches provider

4. **LLM Timeout**
   - Processing exceeds 60 seconds
   - Warning at 45s: "Still processing... (may take up to 60s)"
   - At 60s: Timeout error
   - Message: "Processing timed out. The CV may be too large or complex."
   - Suggestion: "Try a simpler CV or use a faster model."
   - Retry button available

5. **Partial Parsing Results**
   - LLM successfully extracts some but not all fields
   - Results display with warning banner
   - Message: "Some sections could not be parsed. This may be due to unusual CV formatting."
   - Missing sections shown with placeholder: "Not found in document"
   - JSON export includes null values for missing fields
   - Suggestion: "Try reformatting your CV or using a different AI provider."

**Success Metrics**:
- Error resolution rate: >60%
- Users who retry after error: >40%
- Provider fallback success rate: >80%

---

### Journey 3: Developer Integration (API Usage)

1. **API Discovery**
   - Developer reads documentation
   - Finds POST /api/parse endpoint
   - Reviews request/response schema
   - Obtains sample cURL command

2. **First API Call**
   - Developer sends test request with sample CV
   - Receives 200 OK with JSON response
   - Validates response schema
   - Extracts relevant fields for their system

3. **Integration**
   - Implements file upload from their application
   - Forwards to CV Parser API
   - Receives structured JSON
   - Maps to internal data models
   - Displays results to end users

4. **Provider Selection**
   - Developer needs enterprise-grade parsing
   - Configures AWS Bedrock credentials
   - Includes provider parameter in API requests
   - Monitors performance and cost

**Success Metrics**:
- API adoption rate among developers: Track API usage
- Integration time: <2 hours for basic integration
- API error rate: <2%

---

## Success Metrics

### Quantitative Metrics

**Usage Metrics**:
- Daily active users: Target 50+ (Month 1), 200+ (Month 3)
- CVs parsed per day: Target 100+ (Month 1), 500+ (Month 3)
- Average session duration: Target 2-5 minutes
- Bounce rate: Target <40%

**Performance Metrics**:
- Average parsing time: Target <30 seconds (Ollama), <15 seconds (OpenAI)
- Success rate: Target >95%
- Error rate: Target <5%
- Time to first result: Target <60 seconds

**Quality Metrics**:
- Field extraction accuracy: Target >90% (manual validation)
- Complete parsing rate: Target >85% (all major fields extracted)
- User retry rate after errors: Target <20%

### Qualitative Metrics

**User Satisfaction**:
- Net Promoter Score (NPS): Target >40
- User feedback sentiment: Target >80% positive
- Feature request volume: Track top requests
- Return user rate: Target >30%

**System Reliability**:
- Uptime: Target 99.5%
- Provider availability: Track Ollama/OpenAI/Bedrock uptime
- Error recovery success: Target >60%

---

## Release Strategy

### Phase 1: MVP Launch (Sprint 1-2)

**Timeline**: Weeks 1-4
**Goal**: Core functionality with Ollama support

**Features**:
- File upload (PDF/DOCX)
- Text extraction
- Ollama LLM integration
- Basic parsing (all major CV sections)
- Structured results display
- JSON export

**Success Criteria**:
- 10 beta users successfully parse CVs
- <5% error rate
- Average parsing time <45 seconds
- All critical bugs resolved

**Launch Activities**:
- Internal testing with sample CVs
- Bug fixing and polish
- Deploy to https://cvparse.demosrv.uk
- Share with beta users

---

### Phase 2: Multi-Provider Support (Sprint 3-4)

**Timeline**: Weeks 5-8
**Goal**: Add OpenAI and AWS Bedrock providers

**Features**:
- OpenAI integration (GPT-4)
- AWS Bedrock integration (Claude)
- Provider selection UI
- Job title recommendations
- Enhanced error handling

**Success Criteria**:
- 50+ CVs parsed across all providers
- OpenAI/Bedrock success rate >95%
- User can successfully switch providers
- Job recommendations relevant >80% of time

**Launch Activities**:
- Expanded user base to 20-30 users
- Provider performance comparison
- Documentation for API integration
- Marketing push to developers

---

### Phase 3: Polish & Enhancements (Sprint 5+)

**Timeline**: Weeks 9+
**Goal**: Production-ready with nice-to-have features

**Features**:
- Multiple export formats
- CV quality scoring
- Provider comparison mode
- Performance optimizations
- Analytics integration

**Success Criteria**:
- 100+ active users
- <2% error rate
- Sub-20 second parsing (OpenAI)
- 5+ developer integrations

**Launch Activities**:
- Public announcement
- Case studies/testimonials
- SEO optimization
- Community feedback loop

---

## Business Context

### Competitive Landscape

**Existing Solutions**:
- **Affinda CV Parser**: Commercial SaaS, expensive, API-focused
- **Sovren**: Enterprise solution, complex integration, high cost
- **Rezi.ai**: Consumer-focused, subscription model, limited API
- **Custom Python Scripts**: Manual, brittle, not AI-powered

**Our Differentiation**:
- ✅ Open-source friendly (no mandatory costs)
- ✅ Multi-provider flexibility (Ollama, OpenAI, Bedrock)
- ✅ No authentication required (frictionless)
- ✅ Simple, clean UI
- ✅ Developer-friendly API
- ✅ Privacy-first (stateless, no data retention)

### Monetization (Future Consideration)

**Current**: Free and open for all users (no monetization)

**Future Options**:
- **Freemium Model**: Free for Ollama, premium for OpenAI/Bedrock
- **API Licensing**: Charge for commercial API usage
- **Enterprise Hosting**: Managed deployment for companies
- **SaaS Add-ons**: Batch processing, team features, analytics

---

## Risk Assessment

### Technical Risks

**Risk 1: LLM Provider Availability**
- **Impact**: High - Core functionality depends on LLM
- **Mitigation**: Multi-provider support, fallback to alternative
- **Status**: Mitigated in Phase 2

**Risk 2: Parsing Accuracy**
- **Impact**: Medium - Poor results reduce user trust
- **Mitigation**: Prompt engineering, few-shot examples, validation
- **Status**: Ongoing optimization

**Risk 3: Performance (Processing Time)**
- **Impact**: Medium - Slow parsing reduces usability
- **Mitigation**: Provider selection (OpenAI faster), timeout handling
- **Status**: Acceptable with current approach

### Product Risks

**Risk 1: Low Adoption**
- **Impact**: Medium - Product unused
- **Mitigation**: Simple onboarding, marketing to recruiters/developers
- **Status**: To be monitored post-launch

**Risk 2: Privacy Concerns**
- **Impact**: High - CV data is sensitive
- **Mitigation**: No data retention, transparent privacy policy, local processing option (Ollama)
- **Status**: Addressed in architecture

**Risk 3: Competitive Pressure**
- **Impact**: Low - Many alternatives exist
- **Mitigation**: Differentiate on ease-of-use, multi-provider, open-source
- **Status**: Acceptable competitive position

---

## Assumptions & Constraints

### Assumptions

1. Users have access to modern web browsers (Chrome, Firefox, Safari, Edge)
2. CV documents are in readable format (not scanned images without OCR)
3. Ollama instance at http://inference.lan:11434 is reliable and accessible
4. Users understand basic CV structure (if fields are missing, it's the CV's issue, not the tool)
5. Processing time of 10-60 seconds is acceptable to users
6. No authentication is acceptable for this use case

### Constraints

1. **No Database**: Stateless architecture, no persistence
2. **File Size Limit**: 5MB maximum to prevent abuse
3. **Processing Timeout**: 60 seconds maximum
4. **No Batch Processing**: One CV at a time (MVP)
5. **LLM Provider Dependency**: Quality depends on model availability
6. **Budget**: Open-source, minimal infrastructure costs

---

## Appendix

### Glossary

- **CV/Resume**: Curriculum Vitae or Resume document
- **LLM**: Large Language Model (AI for text processing)
- **Ollama**: Open-source LLM runtime for local inference
- **Structured Data**: Organized data with defined schema (vs unstructured text)
- **ATS**: Applicant Tracking System
- **PII**: Personally Identifiable Information

### References

- Ollama Documentation: https://ollama.ai/
- OpenAI API Documentation: https://platform.openai.com/docs
- AWS Bedrock Documentation: https://aws.amazon.com/bedrock/

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 1
**Owner**: Product/Engineering Team
