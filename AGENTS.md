# PDF Merger — AI Agent Documentation

## Project Overview

This is a **single-file web application** that provides client-side PDF merging functionality. The application uses `pdf-merger-js` library via CDN for PDF operations, implemented in a standalone HTML file.

**Key Characteristics:**
- **Language Interface**: Japanese UI with English comments and documentation
- **Architecture**: Pure client-side processing — all PDF operations happen in the browser
- **PDF Engine**: [pdf-merger-js](https://www.npmjs.com/package/pdf-merger-js) v5.1.2 loaded via CDN
- **Privacy-First**: Files are never uploaded to any server

## Technology Stack

| Component | Technology |
|-----------|------------|
| Runtime | Web Browser (modern) |
| Language | HTML5, CSS3, Vanilla JavaScript (ES6+) |
| PDF Processing | pdf-merger-js library (browser build via CDN) |
| Package Manager | Bun (available locally) |

## File Structure

```
pdf-merger/
├── pdf-merger.html    # Main application (single file, self-contained)
├── README.md          # Project name/identifier
├── AGENTS.md          # This file
├── 1.pdf              # Sample/test PDF file
├── 2.pdf              # Sample/test PDF file
├── 3.pdf              # Sample/test PDF file
└── 4.pdf              # Sample/test PDF file
```

> **Note**: This project has no configuration files (no package.json, no pyproject.toml, no build configs) because it loads dependencies via CDN.

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

### 1. CSS Styles (lines 7-306)
- Dark-themed cyberpunk-inspired UI
- Responsive design with CSS variables for theming
- Drag-and-drop styling, animations, progress indicators

### 2. HTML Structure (lines 307-343)
- Header with title and badge
- File drop zone with input
- File list display area
- Merge button and progress UI
- Log output area

### 3. JavaScript (lines 345-350)

**Main modules:**

| Section | Description |
|---------|-------------|
| UI Event Handlers | File input, drag-drop, list reordering |
| PDFMerger Import | Dynamic import from CDN |
| `mergePDFs()` | Main orchestration function using pdf-merger-js |
| UI Glue | Merge button click handler |

## Dependencies

### External Dependencies (CDN)

| Package | Version | CDN URL |
|---------|---------|---------|
| pdf-merger-js | 5.1.2 | `https://cdn.jsdelivr.net/npm/pdf-merger-js@5.1.2/+esm` |

Loaded dynamically via ES module import:
```javascript
const module = await import('https://cdn.jsdelivr.net/npm/pdf-merger-js@5.1.2/+esm');
PDFMerger = module.default || module.PDFMerger || module;
```

### Why pdf-merger-js?

Previously, this project used a custom-built pure JavaScript PDF parser with:
- Manual PDF 1.4 xref table parsing
- Custom ZLIB/DEFLATE decompression (RFC 1950/1951)
- ~800+ lines of complex PDF manipulation code

Now replaced with `pdf-merger-js` which:
- Provides robust PDF merging capabilities
- Handles various PDF versions and edge cases
- Significantly reduces code complexity
- Works in both Node.js and browser environments

## Key Implementation Details

### PDF Merger Usage

```javascript
const merger = new PDFMerger();

// Add PDFs from ArrayBuffer
for (const file of files) {
  const arrayBuffer = await file.arrayBuffer();
  await merger.add(arrayBuffer);
}

// Generate merged PDF
const mergedPdf = await merger.saveAsBuffer();
```

### Page Extraction Strategy

The `pdf-merger-js` library handles:
1. Parsing PDF structure
2. Extracting pages in order
3. Merging pages into a single output document
4. Preserving PDF metadata where possible

## Testing

### Manual Testing Procedure
1. Open `pdf-merger.html` in a browser
2. Drag and drop the sample PDFs (`1.pdf`, `2.pdf`, etc.) into the drop zone
3. Reorder files by dragging list items (if needed)
4. Click "PDFを結合してダウンロード" button
5. Verify the downloaded `merged.pdf` contains all pages in correct order

### Supported PDF Types

Supported by pdf-merger-js:
- ✅ PDF 1.4 and above
- ✅ PDFs with various compression methods
- ✅ Most standard PDF features
- ❌ Encrypted/password-protected PDFs (limited support)

## Development Guidelines

### Code Style
- Use modern ES6+ syntax (const/let, arrow functions, async/await)
- Comments in English, UI text in Japanese
- Functional approach for data transformation

### Adding Features
Since this is a single-file application:
- All new functionality must be added to `pdf-merger.html`
- Keep the self-contained nature — prefer CDN imports over npm packages

### Updating pdf-merger-js

To update the library version:
1. Change the CDN URL in the import statement
2. Test thoroughly with sample PDFs
3. Update this documentation

## Security Considerations

1. **No Server Upload**: All processing is client-side — safe for sensitive documents
2. **CDN Dependency**: Library loaded from jsdelivr CDN — consider SRI (Subresource Integrity) for production
3. **Memory Usage**: Entire PDFs are loaded into memory; very large files may cause issues
4. **XSS Protection**: HTML escaping is applied to filenames (`escHtml()` function)

## Browser Compatibility

Requires modern browsers supporting:
- ES6 Modules and dynamic imports
- Async/await
- `Uint8Array` and `ArrayBuffer`
- `TextEncoder` API
- CSS Custom Properties (variables)

Tested on:
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+

## Known Limitations

1. **CDN Dependency**: Requires internet connection to load pdf-merger-js
2. **Encrypted PDFs**: Password-protected PDFs may not work
3. **File Size**: All processing happens in memory; extremely large PDFs may cause browser memory issues
4. **Font Handling**: Some embedded fonts may not be perfectly preserved in edge cases

## Migration History

### 2026-02-26: Migration to pdf-merger-js

**Before:**
- Pure JavaScript implementation (~1000 lines)
- Custom PDF parser for PDF 1.4
- Manual ZLIB/DEFLATE decompression
- Complex xref table handling

**After:**
- pdf-merger-js v5.1.2 via CDN (~350 lines total)
- Simplified code, better reliability
- Support for wider range of PDF versions
