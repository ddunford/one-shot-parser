# ADR-0003: Document Processing Libraries (PDF & DOCX)

**Status**: Accepted
**Date**: 2025-10-16
**Decision Makers**: Engineering Team

---

## Context

The application must extract text from PDF and DOCX files. We need to choose libraries that:
1. Work in Node.js backend
2. Handle common CV formats reliably
3. Preserve document structure (paragraphs, line breaks)
4. Are actively maintained
5. Have minimal dependencies
6. Process files quickly (<5 seconds for typical CV)

---

## Decision

**PDF Extraction**: `pdf-parse` library
**DOCX Extraction**: `mammoth` library

Both are pure JavaScript, well-maintained, and suitable for CV parsing.

---

## Consequences

### PDF (pdf-parse)

**Positive**:
✅ Pure JavaScript (no native dependencies)
✅ Handles most PDF formats
✅ Extracts text with layout preservation
✅ Returns page count and metadata
✅ Lightweight (~2MB)
✅ Actively maintained

**Negative**:
❌ Limited OCR support (scanned PDFs return minimal text)
❌ Complex multi-column layouts may have ordering issues
❌ Password-protected PDFs not supported

### DOCX (mammoth)

**Positive**:
✅ Clean text extraction
✅ Handles complex Word documents
✅ Preserves paragraph structure
✅ Actively maintained
✅ Simple API

**Negative**:
❌ No support for older .doc format (only .docx)
❌ Limited formatting preservation (text only)
❌ Macros are ignored (security feature)

---

## Alternatives Considered

### PDF Alternatives

**Alternative 1: pdf.js**
- ❌ Designed for browser rendering, heavyweight
- ❌ More complex API for text extraction
- ✅ Better multi-column support

**Alternative 2: pdfium (native)**
- ❌ Requires native compilation
- ❌ Adds deployment complexity
- ✅ Better performance for large files

**Alternative 3: Apache Tika**
- ❌ Requires Java runtime
- ❌ Complex setup
- ✅ Supports more formats

**Decision**: pdf-parse offers best balance of simplicity and capability for CV parsing

### DOCX Alternatives

**Alternative 1: docx4js**
- ❌ Less mature
- ❌ Smaller community
- ✅ More formatting options

**Alternative 2: officeparser**
- ❌ Uses native dependencies
- ❌ More complex setup
- ✅ Supports more Office formats

**Decision**: mammoth is most popular and reliable for DOCX text extraction

---

## Implementation Notes

**PDF Extractor** (src/backend/services/extractors/PDFExtractor.ts):
```typescript
import pdf from 'pdf-parse';

class PDFExtractor implements IDocumentExtractor {
  async extract(buffer: Buffer): Promise<ExtractedText> {
    const data = await pdf(buffer, {
      max: 100  // Max 100 pages (safety limit)
    });

    return {
      text: data.text,
      pageCount: data.numpages,
      wordCount: data.text.split(/\s+/).length,
      metadata: {
        title: data.info?.Title,
        author: data.info?.Author
      }
    };
  }
}
```

**DOCX Extractor** (src/backend/services/extractors/DOCXExtractor.ts):
```typescript
import mammoth from 'mammoth';

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

**Factory Pattern**:
```typescript
class ExtractorFactory {
  getExtractor(mimeType: string): IDocumentExtractor {
    if (mimeType === 'application/pdf') {
      return new PDFExtractor();
    } else if (mimeType === 'application/vnd.openxmlformats-officedocument.wordprocessingml.document') {
      return new DOCXExtractor();
    } else {
      throw new Error(`Unsupported file type: ${mimeType}`);
    }
  }
}
```

---

## Future Enhancements

**OCR Support** (for scanned PDFs):
- Add Tesseract.js for OCR
- Detect image-only PDFs and apply OCR
- Trade-off: Slower processing (10-30s additional)

**.doc Support** (older Word format):
- Add antiword or libreoffice-convert
- Trade-off: Native dependencies required

---

## References

- npm: pdf-parse (https://www.npmjs.com/package/pdf-parse)
- npm: mammoth (https://www.npmjs.com/package/mammoth)
- ARCHITECTURE.md § Document Extractor Service

---

**Related ADRs**:
- ADR-0008: File Upload Handling (validation before extraction)
