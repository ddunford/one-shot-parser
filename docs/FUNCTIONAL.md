# Functional Specification - CV Parser

**Project**: cvparse.demosrv.uk
**Version**: 1.0
**Last Updated**: 2025-10-16

---

## Overview

This document details the functional requirements for each feature of the CV Parser application. Each feature includes user stories, functional requirements, UI requirements, business rules, edge cases, and acceptance criteria.

---

## Feature 1: File Upload System

### User Story

**As a** recruiter or job seeker
**I want to** upload a CV file (PDF or DOCX)
**So that** I can get it parsed by the AI

### Functional Requirements

**FR-1.1**: System shall accept PDF files (.pdf extension)
**FR-1.2**: System shall accept Microsoft Word files (.docx extension)
**FR-1.3**: System shall reject files larger than 5MB
**FR-1.4**: System shall reject files with unsupported extensions
**FR-1.5**: System shall provide drag-and-drop upload capability
**FR-1.6**: System shall provide click-to-browse upload capability
**FR-1.7**: System shall display file name and size after selection
**FR-1.8**: System shall display upload progress during file transfer
**FR-1.9**: System shall clear previous results when new file is uploaded

### UI Requirements

**UI-1.1**: Upload area shall be prominently displayed on landing page
**UI-1.2**: Upload area shall have clear visual affordance (dashed border, upload icon)
**UI-1.3**: Drag-over state shall provide visual feedback (highlight border)
**UI-1.4**: File selection shall show preview card with file details
**UI-1.5**: Error messages shall appear inline near upload area
**UI-1.6**: Progress bar shall be visible during upload
**UI-1.7**: Supported formats shall be listed clearly ("PDF or DOCX, max 5MB")

### Business Rules

**BR-1.1**: Only one file can be uploaded at a time
**BR-1.2**: File must be completely uploaded before parsing can begin
**BR-1.3**: Previous parsing results are discarded when new file is uploaded
**BR-1.4**: File validation occurs on both client (immediate feedback) and server (security)

### Edge Cases

**EC-1.1**: User drags multiple files - only first file is accepted, user notified
**EC-1.2**: User uploads corrupted PDF - extraction fails gracefully with error message
**EC-1.3**: User uploads password-protected PDF - error message prompts to remove password
**EC-1.4**: User uploads DOCX with macros - macros are ignored, text extracted safely
**EC-1.5**: User uploads file with special characters in name - filename sanitized
**EC-1.6**: Network interruption during upload - retry mechanism or clear error message

### Acceptance Criteria

**AC-1.1**: User can drag-and-drop a PDF file to upload area and see file preview
**AC-1.2**: User can click upload area to browse and select DOCX file
**AC-1.3**: User receives clear error message when uploading .txt file
**AC-1.4**: User receives clear error message when uploading 7MB file
**AC-1.5**: Upload progress bar accurately reflects upload progress
**AC-1.6**: File name and size are displayed correctly after selection
**AC-1.7**: Previous results are cleared when new file is uploaded

---

## Feature 2: Document Text Extraction

### User Story

**As a** system component
**I want to** extract text from uploaded PDF and DOCX files
**So that** the text can be sent to the LLM for parsing

### Functional Requirements

**FR-2.1**: System shall extract text from PDF files using pdf-parse or similar library
**FR-2.2**: System shall extract text from DOCX files using mammoth or similar library
**FR-2.3**: System shall preserve paragraph structure and line breaks
**FR-2.4**: System shall handle multi-column PDF layouts
**FR-2.5**: System shall extract text from tables in documents
**FR-2.6**: System shall process extraction in <5 seconds for typical CV (1-3 pages)
**FR-2.7**: System shall return extracted text as plain string
**FR-2.8**: System shall handle extraction errors gracefully

### UI Requirements

**UI-2.1**: Processing indicator shall show "Extracting text from document..."
**UI-2.2**: Estimated time shall be displayed (typically 2-5 seconds)
**UI-2.3**: Spinner or progress animation during extraction
**UI-2.4**: No extracted text shall be displayed to user (internal processing only)

### Business Rules

**BR-2.1**: Extraction must complete successfully before LLM processing begins
**BR-2.2**: Extracted text is not stored or logged (privacy)
**BR-2.3**: Extraction timeout is 10 seconds, after which processing fails
**BR-2.4**: Images in documents are ignored (no OCR in MVP)

### Edge Cases

**EC-2.1**: PDF with scanned images only - extraction returns empty/minimal text, user notified
**EC-2.2**: DOCX with complex formatting (headers, footers) - extracted with best effort
**EC-2.3**: Document with non-English characters - UTF-8 encoding preserves characters
**EC-2.4**: Very long CV (10+ pages) - extraction may be slow, timeout protection applies
**EC-2.5**: Corrupted file structure - extraction library error caught, user sees friendly error

### Acceptance Criteria

**AC-2.1**: System successfully extracts text from sample PDF CV (2 pages, <3 seconds)
**AC-2.2**: System successfully extracts text from sample DOCX CV (3 pages, <3 seconds)
**AC-2.3**: Extracted text preserves paragraph breaks
**AC-2.4**: Multi-column CV layout is extracted in readable order
**AC-2.5**: Table data (education, experience) is extracted correctly
**AC-2.6**: User sees "Extracting..." message during processing
**AC-2.7**: Corrupted file shows error: "Unable to extract text from document"

---

## Feature 3: LLM Integration - Ollama Provider

### User Story

**As a** developer or cost-conscious user
**I want to** use my local Ollama instance for CV parsing
**So that** I can process CVs without API costs

### Functional Requirements

**FR-3.1**: System shall connect to Ollama at http://inference.lan:11434
**FR-3.2**: System shall use Ollama chat API for parsing requests
**FR-3.3**: System shall send structured prompt with CV text
**FR-3.4**: System shall request JSON response format
**FR-3.5**: System shall timeout after 60 seconds if no response
**FR-3.6**: System shall validate JSON response schema
**FR-3.7**: System shall handle Ollama connection failures gracefully
**FR-3.8**: System shall display Ollama provider status (online/offline)

### UI Requirements

**UI-3.1**: Provider dropdown shall list "Ollama (Local)" as default
**UI-3.2**: Ollama status indicator (green dot = online, red = offline)
**UI-3.3**: Processing message: "Analyzing with Ollama..."
**UI-3.4**: Estimated time display: "May take 20-60 seconds"
**UI-3.5**: Progress animation during LLM processing
**UI-3.6**: Connection error shows retry button

### Business Rules

**BR-3.1**: Ollama is the default provider if no other selection is made
**BR-3.2**: Ollama URL is configurable via environment variable
**BR-3.3**: No API key required for Ollama
**BR-3.4**: Processing timeout is 60 seconds (configurable)
**BR-3.5**: Failed requests are logged (no PII) for debugging

### Edge Cases

**EC-3.1**: Ollama server is down - immediate error with suggestion to check server
**EC-3.2**: Ollama is slow (>60s) - timeout error with suggestion to try faster model
**EC-3.3**: Ollama returns invalid JSON - parsing error, suggest retry
**EC-3.4**: Ollama returns partial data - display what's available, flag missing fields
**EC-3.5**: Network unreachable - clear error message about connectivity

### Acceptance Criteria

**AC-3.1**: System successfully connects to Ollama at http://inference.lan:11434
**AC-3.2**: Sample CV is parsed in <45 seconds with Ollama
**AC-3.3**: Parsed JSON matches expected schema
**AC-3.4**: Ollama offline status is detected and displayed
**AC-3.5**: Connection failure shows error: "Cannot connect to Ollama. Ensure it's running."
**AC-3.6**: User can retry failed request
**AC-3.7**: Timeout after 60 seconds shows error with guidance

---

## Feature 4: Structured CV Data Parsing

### User Story

**As a** recruiter
**I want** the system to extract structured information from CVs
**So that** I can quickly assess candidate qualifications

### Functional Requirements

**FR-4.1**: System shall extract personal information (name, email, phone, address)
**FR-4.2**: System shall categorize skills into: technical, soft, languages, certifications
**FR-4.3**: System shall parse education history with: institution, degree, field, dates
**FR-4.4**: System shall parse work experience with: company, title, dates, responsibilities
**FR-4.5**: System shall identify current employment (boolean flag)
**FR-4.6**: System shall extract optional fields: LinkedIn, GitHub, personal website
**FR-4.7**: System shall handle missing fields gracefully (null or empty array)
**FR-4.8**: System shall output data in consistent JSON schema

### UI Requirements

**UI-4.1**: Results shall be organized into clear sections: Personal, Skills, Education, Experience
**UI-4.2**: Personal info displayed in card format at top
**UI-4.3**: Skills displayed as categorized tags/pills
**UI-4.4**: Education displayed in chronological timeline
**UI-4.5**: Experience displayed in reverse chronological order
**UI-4.6**: Missing fields show "Not found" or hidden from display
**UI-4.7**: Dates formatted consistently (YYYY-MM or "Present")

### Business Rules

**BR-4.1**: All fields are optional (missing data is acceptable)
**BR-4.2**: Skills must be categorized; uncategorized skills go to "Other"
**BR-4.3**: Dates must be parsed as YYYY-MM format or null
**BR-4.4**: Email and phone are validated for format (basic regex)
**BR-4.5**: Experience entries sorted by start date (descending)
**BR-4.6**: Current job indicated by null/empty end date or "Present"

### Edge Cases

**EC-4.1**: CV with no skills section - skills array is empty, displayed as "No skills found"
**EC-4.2**: CV with ambiguous dates - LLM uses best guess, may be incorrect
**EC-4.3**: CV with non-standard format - some fields may be missed
**EC-4.4**: CV with multiple phone numbers - only first number extracted
**EC-4.5**: CV in non-English language - parsing quality depends on LLM multilingual support
**EC-4.6**: CV with overlapping job dates - displayed as-is, not validated

### Acceptance Criteria

**AC-4.1**: Sample CV #1 extracts all personal info fields correctly (name, email, phone, address)
**AC-4.2**: Sample CV #2 categorizes 10 skills correctly (5 technical, 3 soft, 2 languages)
**AC-4.3**: Sample CV #3 extracts 3 education entries with complete data
**AC-4.4**: Sample CV #4 extracts 5 work experiences in reverse chronological order
**AC-4.5**: CV missing phone number shows "Not found" or no phone section
**AC-4.6**: Current job is identified with "Present" end date
**AC-4.7**: JSON output validates against schema (automated test)

---

## Feature 5: Results Display Interface

### User Story

**As a** user
**I want to** see parsed CV data in a clean, organized format
**So that** I can quickly understand the candidate's profile

### Functional Requirements

**FR-5.1**: System shall display personal information in dedicated section
**FR-5.2**: System shall display skills grouped by category
**FR-5.3**: System shall display education in timeline format
**FR-5.4**: System shall display work experience in reverse chronological timeline
**FR-5.5**: System shall highlight current employment
**FR-5.6**: System shall be responsive on mobile, tablet, and desktop
**FR-5.7**: System shall scroll to results after parsing completes
**FR-5.8**: System shall allow re-parsing with different provider

### UI Requirements

**UI-5.1**: Personal info card with avatar placeholder and contact details
**UI-5.2**: Skills section with color-coded category tags
**UI-5.3**: Education timeline with institution logos/icons
**UI-5.4**: Experience timeline with company, role, and bullet points
**UI-5.5**: "Current" badge on current job
**UI-5.6**: Clean, modern design with good typography
**UI-5.7**: Adequate spacing between sections
**UI-5.8**: Print-friendly layout (optional)

### Business Rules

**BR-5.1**: Results are displayed only after successful parsing
**BR-5.2**: Partial results are acceptable (display what's available)
**BR-5.3**: Results persist until new file is uploaded
**BR-5.4**: Results are not saved server-side (client-side only)

### Edge Cases

**EC-5.1**: Very long CV (20+ skills, 10+ jobs) - scrollable sections or pagination
**EC-5.2**: No profile picture in CV - generic avatar placeholder used
**EC-5.3**: Extremely short CV (1 job, 2 skills) - still displays in full layout
**EC-5.4**: Wide variety of date formats - normalized to YYYY-MM display

### Acceptance Criteria

**AC-5.1**: Results page displays immediately after parsing completes
**AC-5.2**: All four major sections are visible (Personal, Skills, Education, Experience)
**AC-5.3**: Skills are color-coded by category
**AC-5.4**: Education and experience are in chronological order
**AC-5.5**: Layout is responsive on mobile (320px width)
**AC-5.6**: Layout is readable on desktop (1920px width)
**AC-5.7**: User can scroll through results smoothly
**AC-5.8**: "Parse Another CV" button is visible at bottom

---

## Feature 6: JSON Export

### User Story

**As a** developer or power user
**I want to** copy the parsed CV data as JSON
**So that** I can integrate it into my own systems

### Functional Requirements

**FR-6.1**: System shall display parsed data as pretty-printed JSON
**FR-6.2**: System shall provide one-click copy to clipboard functionality
**FR-6.3**: System shall show copy success confirmation
**FR-6.4**: System shall syntax-highlight JSON for readability
**FR-6.5**: System shall validate JSON is well-formed before display
**FR-6.6**: System shall include all parsed fields in JSON (including null values)

### UI Requirements

**UI-6.1**: Expandable "View JSON" section below results
**UI-6.2**: JSON displayed in monospace font with syntax highlighting
**UI-6.3**: "Copy to Clipboard" button prominently placed
**UI-6.4**: Success message/toast on copy: "Copied to clipboard!"
**UI-6.5**: Expand/collapse animation for JSON section
**UI-6.6**: JSON is scrollable if very long

### Business Rules

**BR-6.1**: JSON output must validate against defined schema
**BR-6.2**: JSON includes all fields even if null (for consistency)
**BR-6.3**: JSON is indented with 2 spaces for readability
**BR-6.4**: Copy function works on all modern browsers

### Edge Cases

**EC-6.1**: Browser blocks clipboard access - fallback to "Select All" hint
**EC-6.2**: Very large JSON (>100KB) - still fully displayed, may be slow
**EC-6.3**: Special characters in JSON strings - properly escaped
**EC-6.4**: User clicks copy multiple times - each click triggers success message

### Acceptance Criteria

**AC-6.1**: JSON section is collapsed by default
**AC-6.2**: Clicking "View JSON" expands section and shows formatted JSON
**AC-6.3**: JSON is syntax-highlighted (keys, strings, values in different colors)
**AC-6.4**: Clicking "Copy to Clipboard" copies full JSON
**AC-6.5**: Success message appears for 2 seconds after copy
**AC-6.6**: Copied JSON can be pasted into text editor and is valid
**AC-6.7**: JSON schema validation passes (automated test)

---

## Feature 7: Multi-Provider Support (Post-MVP)

### User Story

**As a** user seeking high-quality results
**I want to** choose between Ollama, OpenAI, and AWS Bedrock
**So that** I can use the best AI model for my needs

### Functional Requirements

**FR-7.1**: System shall support provider selection via dropdown
**FR-7.2**: System shall integrate with OpenAI API (GPT-4 and GPT-3.5-turbo)
**FR-7.3**: System shall integrate with AWS Bedrock (Claude models)
**FR-7.4**: System shall use identical prompt structure across providers
**FR-7.5**: System shall enforce same JSON schema across providers
**FR-7.6**: System shall handle provider-specific errors gracefully
**FR-7.7**: System shall display provider status for each option
**FR-7.8**: System shall store API keys securely (environment variables)

### UI Requirements

**UI-7.1**: Provider dropdown with 3 options: Ollama, OpenAI, AWS Bedrock
**UI-7.2**: Status indicator (online/offline/needs config) for each provider
**UI-7.3**: Tooltip explaining each provider's benefits
**UI-7.4**: Selected provider highlighted in UI during processing
**UI-7.5**: Provider-specific error messages (e.g., "OpenAI API key invalid")

### Business Rules

**BR-7.1**: API keys are never exposed to client-side code
**BR-7.2**: Provider selection persists for session (not across page refresh)
**BR-7.3**: If provider is unavailable, user is prompted to select alternative
**BR-7.4**: All providers must return same JSON schema structure

### Edge Cases

**EC-7.1**: OpenAI API key not configured - provider shows "Needs Configuration"
**EC-7.2**: AWS Bedrock region unavailable - error with region suggestion
**EC-7.3**: User switches provider mid-processing - current request is cancelled
**EC-7.4**: All providers offline - user sees comprehensive error message

### Acceptance Criteria

**AC-7.1**: Dropdown shows 3 providers with correct labels
**AC-7.2**: Selecting OpenAI uses gpt-4 model successfully
**AC-7.3**: Selecting Bedrock uses Claude model successfully
**AC-7.4**: Same CV parsed with all 3 providers yields similar results
**AC-7.5**: Invalid API key shows clear error message
**AC-7.6**: Provider status indicators reflect actual availability

---

## Feature 8: Job Title Recommendations (Post-MVP)

### User Story

**As a** job seeker
**I want to** receive job title recommendations based on my CV
**So that** I can target appropriate roles in my job search

### Functional Requirements

**FR-8.1**: System shall generate 3-5 job title recommendations
**FR-8.2**: Recommendations shall be based on skills, experience, and seniority
**FR-8.3**: System shall provide brief reasoning for each recommendation
**FR-8.4**: System shall use LLM to generate recommendations (same request as parsing)
**FR-8.5**: System shall handle cases where recommendations can't be generated

### UI Requirements

**UI-8.1**: Recommendations displayed in dedicated section after experience
**UI-8.2**: Each recommendation shown as card with title and reasoning
**UI-8.3**: Recommendations styled distinctly (e.g., blue accent color)
**UI-8.4**: "Why this role?" explanation under each title
**UI-8.5**: Optional: Confidence indicator (high/medium/low)

### Business Rules

**BR-8.1**: Recommendations are generated in same LLM call as parsing (efficiency)
**BR-8.2**: Minimum 3 recommendations, maximum 5
**BR-8.3**: Recommendations are sorted by relevance (best first)
**BR-8.4**: If LLM can't generate recommendations, section is hidden

### Edge Cases

**EC-8.1**: Very junior CV (no experience) - entry-level roles recommended
**EC-8.2**: Career changer CV (mixed skills) - transitional roles suggested
**EC-8.3**: Executive CV (C-level) - senior/leadership roles recommended
**EC-8.4**: Unclear career path - broader role categories suggested

### Acceptance Criteria

**AC-8.1**: Sample senior developer CV generates 4 relevant job titles
**AC-8.2**: Each recommendation includes reasoning (1-2 sentences)
**AC-8.3**: Recommendations match candidate's seniority level
**AC-8.4**: Recommendations section appears after work experience
**AC-8.5**: If no recommendations, section is hidden (no error shown)

---

## Feature 9: Error Handling & User Feedback

### User Story

**As a** user
**I want to** receive clear feedback when errors occur
**So that** I know what went wrong and how to fix it

### Functional Requirements

**FR-9.1**: System shall display specific error messages for each error type
**FR-9.2**: System shall provide actionable guidance in error messages
**FR-9.3**: System shall offer retry button for transient errors
**FR-9.4**: System shall log errors server-side (no PII) for debugging
**FR-9.5**: System shall handle partial parsing results gracefully
**FR-9.6**: System shall time out long-running requests
**FR-9.7**: System shall recover from errors without page refresh

### UI Requirements

**UI-9.1**: Error messages displayed in red banner at top of page
**UI-9.2**: Error icon (⚠️) accompanies error message
**UI-9.3**: Retry button visible for retryable errors
**UI-9.4**: Troubleshooting tips expandable in error banner
**UI-9.5**: Partial results show warning (⚠️ Some sections could not be parsed)
**UI-9.6**: Errors are dismissible (X button)

### Business Rules

**BR-9.1**: Client-side validation occurs before server request
**BR-9.2**: Server-side validation is authoritative
**BR-9.3**: Errors are categorized: validation, processing, timeout, network
**BR-9.4**: User can retry up to 3 times before being asked to contact support

### Error Types & Messages

**ET-9.1 - Invalid File Type**:
- Message: "Unsupported file format. Please upload a PDF or DOCX file."
- Action: User selects different file

**ET-9.2 - File Too Large**:
- Message: "File size exceeds 5MB limit. Please compress your document or remove images."
- Action: User uploads smaller file

**ET-9.3 - Extraction Failed**:
- Message: "Unable to extract text from document. The file may be corrupted or password-protected."
- Action: User tries different file or removes password

**ET-9.4 - LLM Timeout**:
- Message: "Processing timed out after 60 seconds. Try using a faster AI provider (OpenAI) or a simpler CV."
- Action: Retry button or provider selection

**ET-9.5 - LLM Connection Failed**:
- Message: "Cannot connect to Ollama at http://inference.lan:11434. Ensure Ollama is running."
- Action: Start Ollama or select different provider

**ET-9.6 - Invalid Response**:
- Message: "AI returned invalid data. This may be a temporary issue. Please try again."
- Action: Retry button

**ET-9.7 - Network Error**:
- Message: "Network connection lost. Check your internet and try again."
- Action: Retry button

### Edge Cases

**EC-9.1**: Multiple errors occur - only most recent error displayed
**EC-9.2**: Error during retry - error message updates with retry count
**EC-9.3**: Partial parsing with error - both results and error shown
**EC-9.4**: User dismisses error then retries - error may reappear

### Acceptance Criteria

**AC-9.1**: Invalid file type shows error message within 1 second
**AC-9.2**: File too large shows error immediately after selection
**AC-9.3**: LLM timeout (after 60s) shows clear error with suggestions
**AC-9.4**: Connection failure shows error within 5 seconds
**AC-9.5**: Retry button successfully retries failed request
**AC-9.6**: Partial results display with warning banner
**AC-9.7**: Error banner can be dismissed by user
**AC-9.8**: All error messages are user-friendly (no technical jargon)

---

## Data Schema Definition

### CV Data JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["personalInfo", "skills", "education", "experience"],
  "properties": {
    "personalInfo": {
      "type": "object",
      "properties": {
        "name": { "type": ["string", "null"] },
        "email": { "type": ["string", "null"], "format": "email" },
        "phone": { "type": ["string", "null"] },
        "address": { "type": ["string", "null"] },
        "linkedin": { "type": ["string", "null"], "format": "uri" },
        "github": { "type": ["string", "null"], "format": "uri" },
        "website": { "type": ["string", "null"], "format": "uri" }
      }
    },
    "summary": { "type": ["string", "null"] },
    "skills": {
      "type": "object",
      "properties": {
        "technical": { "type": "array", "items": { "type": "string" } },
        "soft": { "type": "array", "items": { "type": "string" } },
        "languages": { "type": "array", "items": { "type": "string" } },
        "certifications": { "type": "array", "items": { "type": "string" } }
      }
    },
    "education": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "institution": { "type": "string" },
          "degree": { "type": "string" },
          "field": { "type": "string" },
          "startDate": { "type": ["string", "null"], "pattern": "^\\d{4}-\\d{2}$" },
          "endDate": { "type": ["string", "null"], "pattern": "^(\\d{4}-\\d{2}|Present)$" },
          "gpa": { "type": ["string", "null"] }
        },
        "required": ["institution", "degree"]
      }
    },
    "experience": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "company": { "type": "string" },
          "title": { "type": "string" },
          "location": { "type": ["string", "null"] },
          "startDate": { "type": ["string", "null"], "pattern": "^\\d{4}-\\d{2}$" },
          "endDate": { "type": ["string", "null"], "pattern": "^(\\d{4}-\\d{2}|Present|null)$" },
          "current": { "type": "boolean" },
          "responsibilities": { "type": "array", "items": { "type": "string" } },
          "achievements": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["company", "title"]
      }
    },
    "recommendations": {
      "type": "object",
      "properties": {
        "jobTitles": { "type": "array", "items": { "type": "string" } },
        "reasoning": { "type": "string" }
      }
    }
  }
}
```

---

## Non-Functional Requirements

### Performance

**NFR-P1**: File upload shall complete in <5 seconds for 5MB file on broadband connection
**NFR-P2**: Text extraction shall complete in <5 seconds for typical CV (1-3 pages)
**NFR-P3**: LLM parsing shall complete in <60 seconds (timeout threshold)
**NFR-P4**: Results display shall render in <2 seconds after parsing completes
**NFR-P5**: Page load time shall be <3 seconds on broadband connection

### Usability

**NFR-U1**: User shall be able to complete primary journey (upload to results) in <5 clicks
**NFR-U2**: Error messages shall be understandable by non-technical users
**NFR-U3**: UI shall be accessible (WCAG 2.1 Level AA minimum)
**NFR-U4**: Application shall work on Chrome, Firefox, Safari, Edge (last 2 versions)
**NFR-U5**: Mobile responsiveness for screen widths 320px to 1920px

### Security

**NFR-S1**: Uploaded files shall not be stored on server after processing
**NFR-S2**: Extracted text shall not be logged (contains PII)
**NFR-S3**: API keys shall never be exposed to client-side code
**NFR-S4**: File upload shall enforce 5MB limit server-side (not just client)
**NFR-S5**: File type validation shall occur server-side (not just client)

### Reliability

**NFR-R1**: System shall handle up to 100 concurrent requests
**NFR-R2**: System uptime target: 99.5%
**NFR-R3**: Failed LLM requests shall not crash application
**NFR-R4**: System shall recover from errors without page refresh

### Maintainability

**NFR-M1**: Code shall be documented with clear comments
**NFR-M2**: API endpoints shall follow RESTful conventions
**NFR-M3**: JSON schema shall be versioned and validated
**NFR-M4**: Logging shall be sufficient for debugging production issues

---

## Appendix: Sample Test Cases

### TC-1: Happy Path - PDF Upload and Parse

1. Navigate to https://cvparse.demosrv.uk
2. Drag sample-cv.pdf (2MB, 2 pages) to upload area
3. Verify file preview shows "sample-cv.pdf, 2.1 MB"
4. Keep default provider (Ollama)
5. Click "Parse CV" button
6. Observe "Extracting text..." message
7. Observe "Analyzing with Ollama..." message
8. Wait for results (expected: 20-45 seconds)
9. Verify all sections appear: Personal Info, Skills, Education, Experience
10. Verify personal info extracted correctly (name, email, phone)
11. Verify 8+ skills categorized correctly
12. Verify 2 education entries with dates
13. Verify 3 experience entries in reverse chronological order
14. Click "View JSON" to expand JSON section
15. Click "Copy to Clipboard" button
16. Verify success message appears
17. Paste into text editor and verify valid JSON

**Expected Result**: All steps pass, data is accurate and complete

---

### TC-2: Error Handling - Invalid File Type

1. Navigate to https://cvparse.demosrv.uk
2. Attempt to upload sample-cv.txt file
3. Verify error message: "Unsupported file format. Please upload PDF or DOCX."
4. Verify upload area highlights in red
5. Verify Parse button is disabled
6. Upload sample-cv.pdf instead
7. Verify error clears and parse button enables

**Expected Result**: Clear error message, user can recover

---

### TC-3: LLM Provider Switching

1. Navigate to https://cvparse.demosrv.uk
2. Upload sample-cv.pdf
3. Select "OpenAI" from provider dropdown
4. Click "Parse CV"
5. Observe parsing with OpenAI (faster, expected <15 seconds)
6. Verify results appear
7. Click "Parse Again" button
8. Select "Ollama" from provider dropdown
9. Click "Parse CV" again
10. Observe parsing with Ollama
11. Compare results (should be similar but not identical)

**Expected Result**: Both providers work, results are comparable

---

**Document Status**: ✅ Complete
**Next Review Date**: Post-Sprint 1
**Owner**: Engineering Team
