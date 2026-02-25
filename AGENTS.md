# PDF Merger — AI Agent Documentation

## Project Overview

This is a **single-file web application** that provides client-side PDF merging functionality. The application is implemented entirely in pure JavaScript without any external dependencies - a full scratch implementation.

**Key Characteristics:**
- **Language Interface**: Japanese UI with English comments and documentation
- **Architecture**: Pure client-side processing — all PDF operations happen in the browser
- **PDF Engine**: Full scratch PDF parser and builder (no external libraries)
- **Privacy-First**: Files are never uploaded to any server

## Technology Stack

| Component | Technology |
|-----------|------------|
| Runtime | Web Browser (modern) |
| Language | HTML5, CSS3, Vanilla JavaScript (ES6+) |
| PDF Processing | Custom-built PDF parser and merger |
| Compression | Pure JS implementation of ZLIB/DEFLATE (RFC 1950/1951) |
| Package Manager | Bun (available locally for development) |

## File Structure

```
pdf-merger/
├── pdf-merger.html       # Main application (single file, self-contained)
├── IMPLEMENTATION_PLAN.md # Implementation plan document
├── README.md             # Project name/identifier
├── AGENTS.md             # This file
├── 1.pdf                 # Sample/test PDF file
├── 2.pdf                 # Sample/test PDF file
├── 3.pdf                 # Sample/test PDF file
└── 4.pdf                 # Sample/test PDF file
```

> **Note**: This project has no configuration files (no package.json, no pyproject.toml, no build configs) because it is a pure HTML/JS application.

## How to Run

Simply open `pdf-merger.html` in any modern web browser:

```bash
# Option 1: Direct file open
open pdf-merger.html

# Option 2: Serve via local HTTP server (recommended for testing)
bun -p 8000
# Then navigate to http://localhost:8000/pdf-merger.html
```

No build step, no installation, no dependencies to fetch.

## Code Organization

The `pdf-merger.html` file contains three main sections:

### 1. CSS Styles
- Dark-themed cyberpunk-inspired UI
- Responsive design with CSS variables for theming
- Drag-and-drop styling, animations, progress indicators

### 2. HTML Structure
- Header with title and badge
- File drop zone with input
- File list display area
- Merge button and progress UI
- Log output area

### 3. JavaScript (5 main sections)

| Section | Description |
|---------|-------------|
| UI Event Handlers | File input, drag-drop, list reordering |
| PDF Parser | XREF table parsing, object extraction |
| ZLIB/DEFLATE | RFC 1950/1951 decompression for FlateDecode |
| PDF Merge Logic | Page collection, stream processing, PDF building |
| Main Entry Point | Merge button handler, download generation |

## Key Implementation Details

### PDF Parser Capabilities
- Supports **PDF 1.4** format with traditional cross-reference tables
- Parses object offsets from `xref` tables
- Extracts trailer dictionary for `/Root`, `/Info` references
- Fallback linear scan for corrupted/malformed PDFs
- **Limitation**: Does not support PDF 1.5+ cross-reference streams

### DEFLATE/ZLIB Implementation
- Full RFC 1950 (ZLIB) and RFC 1951 (DEFLATE) implementation
- Supports all block types: uncompressed, fixed Huffman, dynamic Huffman
- Uses flat-array Huffman lookup tables for performance
- Required for decompressing FlateDecode streams in PDF content

### Page Extraction Strategy
1. Scans all objects to find `/Type /Page` entries
2. Attempts to follow `/Pages` → `/Kids` ordering for correct page sequence
3. Falls back to object number sorting if Kids array unavailable
4. Copies content streams and resources with remapped object numbers

### PDF Building Process
1. Creates new PDF with version header (%PDF-1.4)
2. Assigns new object numbers for Catalog, Pages, and page objects
3. Decompresses content streams, combines them, and stores uncompressed
4. Rebuilds XREF table with correct byte offsets
5. Outputs final PDF with proper trailer

## Testing

### Manual Testing Procedure
1. Open `pdf-merger.html` in a browser
2. Drag and drop the sample PDFs (`1.pdf`, `2.pdf`, etc.) into the drop zone
3. Reorder files by dragging list items (if needed)
4. Click "PDFを結合してダウンロード" button
5. Verify the downloaded `merged.pdf` contains all pages in correct order

### Supported PDF Types
- ✅ PDF 1.4 with traditional xref tables
- ✅ PDFs with FlateDecode compressed streams
- ⚠️ PDF 1.5+ with cross-reference streams (limited support, may use fallback)
- ❌ Encrypted/password-protected PDFs (not supported)

## Development Guidelines

### Code Style
- Use modern ES6+ syntax (const/let, arrow functions, classes, async/await)
- Comments in English, UI text in Japanese
- Functional approach for data transformation
- Chunked processing for large arrays to avoid stack overflow

### Implementation Approach
The project follows a full scratch implementation strategy:
1. **No external libraries** for PDF processing
2. **Hand-coded parsers** for PDF structure and compression
3. **Manual memory management** considerations for large files

### Modifying the PDF Parser
The PDF parsing logic is tightly coupled to the PDF 1.4 specification:
- `PDFParser.parseXref()` — handles traditional xref table format
- `PDFParser.rebuildXref()` — fallback for missing/corrupted xref
- Object byte offsets are stored in a `Map<number, number>`

## Security Considerations

1. **No Server Upload**: All processing is client-side — safe for sensitive documents
2. **No External Resources**: No CDN, no external scripts — no supply chain risks
3. **Memory Usage**: Entire PDFs are loaded into memory; very large files may cause issues
4. **XSS Protection**: HTML escaping is applied to filenames (`escHtml()` function)

## Browser Compatibility

Requires modern browsers supporting:
- ES6 Classes and arrow functions
- `Uint8Array` and `ArrayBuffer`
- `TextEncoder` API
- CSS Custom Properties (variables)
- ES6 Module syntax (for development)

Tested on:
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+

## Known Limitations

1. **XRef Streams**: PDF 1.5+ cross-reference streams are not fully supported
2. **Object Streams**: Compressed object streams (common in PDF 1.5+) not supported
3. **Encryption**: Encrypted PDFs cannot be processed
4. **Complex Resources**: Embedded fonts and complex resource hierarchies may not be perfectly preserved
5. **File Size**: All processing happens in memory; extremely large PDFs may cause browser memory issues

## Project History

### 2026-02-26: Full Scratch Implementation

**Initial Approach:**
- Started with pure JavaScript implementation (~1000 lines)
- Custom PDF parser for PDF 1.4
- Manual ZLIB/DEFLATE decompression
- Complex xref table handling

**Temporary Migration (reverted):**
- Briefly migrated to `pdf-merger-js` library via CDN
- Reduced code complexity but introduced external dependency
- **Reverted** back to full scratch implementation

**Final Implementation:**
- Full scratch pure JavaScript implementation
- No external dependencies
- Complete control over PDF processing pipeline
- Japanese UI maintained throughout
