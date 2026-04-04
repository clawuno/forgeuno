---
name: office-pdf
display_name: Office PDF
version: "1.0.1"
description: >-
  Read and generate PDF files. Read: extract text, parse tables, analyze
  structure from existing PDFs. Generate: create PDFs from Markdown, HTML,
  or structured content with professional design. Also fill PDF forms.
  Use this skill whenever the task involves reading from or creating PDF files.
  Keywords: PDF, read PDF, extract text, parse table, generate PDF, export PDF,
  Markdown to PDF, convert to PDF, report, briefing, invoice, form.
tags:
  - pdf
  - document
  - extract
  - generate
  - markdown
allowed-tools: >-
  Read Write Edit Bash
---

# Office PDF

Read, generate, and process PDF files. Covers: extracting text and tables from
existing PDFs, converting Markdown to styled PDF, creating professional documents
(reports, proposals, invoices), and filling PDF forms.

---

## Task Routing

| Task | Route | Method |
|------|-------|--------|
| Create new PDF from scratch | **CREATE** | Dual-engine: HTML cover + ReportLab body |
| Convert Markdown to PDF | **MD-TO-PDF** | Markdown → styled HTML → Chrome printToPDF |
| Read/extract text from PDF | **READ** | pdfplumber (tables), pypdf (text) |
| Fill PDF form fields | **FILL** | pypdf form filling |
| Reformat existing content as PDF | **REFORMAT** | Extract → restructure → CREATE |

---

## Route: CREATE

### Dual-Engine Architecture

PDFs with covers benefit from two rendering engines:

| Part | Engine | Why |
|------|--------|-----|
| **Cover page** | HTML+CSS → Chrome/Playwright → PDF | CSS natively supports gradients, grids, blend modes, custom fonts — hard with PDF drawing APIs |
| **Body** | ReportLab | Stable paragraph flow, pagination, headers/footers, page numbers |
| **Merge** | pypdf | Combine cover.pdf + body.pdf into final.pdf |

If Chrome/Playwright is unavailable, fall back to ReportLab for covers (simpler design, but still functional).

### Workflow

1. **Determine document type** — Select visual identity and cover pattern
2. **Generate cover** — HTML+CSS or ReportLab
3. **Generate body** — ReportLab with content blocks
4. **Merge** — pypdf concatenation
5. **Validate** — Render pages to PNG, visual inspection

### 15 Document Types

Each type has a distinct visual identity (cover pattern, fonts, accent color):

| Type | Cover Style | Best For |
|------|-------------|----------|
| Report | Bold title, date banner | Business reports, quarterly reviews |
| Proposal | Logo-centric, value proposition | Sales proposals, partnerships |
| Whitepaper | Academic, abstract box | Technical deep-dives |
| Manual | Section tabs, version badge | User guides, SOPs |
| Invoice | Compact header, line items | Billing, quotes |
| Certificate | Ornamental border, seal area | Awards, credentials |
| Newsletter | Multi-column, headline | Internal comms, updates |
| Resume | Sidebar + main content | CVs, professional profiles |
| Presentation | Full-bleed images, large type | Handouts, summaries |
| Contract | Numbered clauses, signature blocks | Legal, agreements |
| Research | Abstract + keywords + citation | Academic papers |
| Catalog | Grid layout, product cards | Product listings |
| Brochure | Tri-fold layout, visual-heavy | Marketing materials |
| Menu | Category sections, pricing | Restaurant, services |
| Letterhead | Header + footer branding | Official correspondence |

### Accent Color Selection by Industry

| Industry | Recommended Accent | Hex |
|----------|--------------------|-----|
| Finance/Legal | Navy blue | `#1B365D` |
| Healthcare | Teal | `#008080` |
| Technology | Electric blue | `#0066FF` |
| Education | Forest green | `#228B22` |
| Creative | Coral | `#FF6B6B` |
| Government | Dark red | `#8B0000` |
| Consulting | Charcoal + gold | `#333333` / `#D4AF37` |

### Content Block Types

Structure document body using these block types:

| Block | Description |
|-------|-------------|
| **heading** | Section/subsection titles (H1-H4) |
| **paragraph** | Body text with optional indent |
| **bullet_list** | Unordered list items |
| **numbered_list** | Ordered list items |
| **table** | Row/column data with header row |
| **image** | Embedded image with caption |
| **code_block** | Monospace code with syntax highlighting |
| **blockquote** | Indented quotation with attribution |
| **callout** | Highlighted box (info, warning, tip) |
| **chart** | Bar, line, pie rendered as image |
| **math** | LaTeX formula rendered as image |
| **page_break** | Force new page |
| **horizontal_rule** | Section separator |
| **footnote** | Numbered reference at page bottom |

---

## Route: MD-TO-PDF

Convert Markdown files to professionally typeset PDFs using pure Python (ReportLab). No browser needed.

```bash
python3 scripts/md2pdf.py --input report.md --output report.pdf --title "My Report" --theme corporate
```

### Features

- **CJK mixed text**: Automatic character-level font switching (Chinese/Japanese/Korean + Latin)
- **Code blocks**: Preserved indentation and line breaks, auto-truncation at 30 lines
- **Tables**: Smart proportional column widths
- **Full document structure**: Cover → frontispiece → TOC → body → back cover
- **PDF bookmarks**: Clickable sidebar navigation from headings
- **Page numbers, headers, footers**: Running chapter title in header
- **Watermark support**
- **10 design themes**

### Available Themes

| Theme | Personality |
|-------|-------------|
| `corporate` | Professional blue, clean lines |
| `academic` | Formal, serif, dense |
| `modern` | Bold gradients, sans-serif |
| `minimal` | White space, understated |
| `creative` | Vibrant accents, playful |
| `nature` | Earth tones, organic |
| `elegant` | Serif, gold accents |
| `tech` | Dark theme, monospace accents |
| `magazine` | Editorial layout, pull quotes |
| `chinese_formal` | GB/T compliant, SimHei/FangSong |

### Key Options

```bash
python3 md2pdf.py \
  --input report.md \
  --output report.pdf \
  --title "Quarterly Report" \
  --subtitle "Q1 2026" \
  --author "Research Team" \
  --theme corporate \
  --toc \                    # Generate table of contents
  --watermark "DRAFT" \      # Add watermark
  --header-title "Q1 Report" # Running header
```

### Dependencies

`pip install reportlab` — 5MB, pure Python, zero C dependencies, zero browser. Works everywhere including sandboxed environments.

---

## Route: READ

```python
# Text extraction
from pypdf import PdfReader
reader = PdfReader("file.pdf")
for page in reader.pages:
    print(page.extract_text())

# Table extraction
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            for row in table:
                print(row)
```

---

## Route: FILL (Forms)

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("form.pdf")
writer = PdfWriter()
writer.append(reader)

# Fill form fields
writer.update_page_form_field_values(
    writer.pages[0],
    {"field_name": "value", "date_field": "2026-04-01"}
)

with open("filled.pdf", "wb") as f:
    writer.write(f)
```

---

## Typography Standards

| Element | Font | Size | Weight |
|---------|------|------|--------|
| Title | Sans-serif (Helvetica/Arial) | 24-32pt | Bold |
| Subtitle | Sans-serif | 14-18pt | Regular |
| H1 | Sans-serif | 18-20pt | Bold |
| H2 | Sans-serif | 14-16pt | Bold |
| Body | Serif (Times/Georgia) or Sans | 10-12pt | Regular |
| Caption | Sans-serif | 8-9pt | Italic |
| Code | Monospace (Courier) | 9-10pt | Regular |
| Footer | Sans-serif | 8pt | Regular |

**Line spacing**: 1.2-1.5× font size for body text.
**Margins**: Print-ready = 1 inch (72pt) all sides. Digital = 0.75 inch.
**Page size**: A4 (595×842pt) for international, Letter (612×792pt) for US.

---

## Validation Checklist

- [ ] PDF opens without errors in all viewers (Preview, Chrome, Acrobat)
- [ ] Cover page renders correctly (images, fonts, colors)
- [ ] Page numbers are sequential and correctly positioned
- [ ] Headers/footers appear on correct pages
- [ ] Tables don't split awkwardly across pages
- [ ] Images are sharp (min 150 DPI for print, 72 DPI for screen)
- [ ] Text is selectable (not rasterized)
- [ ] Fonts are embedded or universally available
- [ ] File size is reasonable (< 10MB for typical documents)

### Preview Command

```bash
# PDF → PNG pages for visual review
pdftoppm -png -r 150 output.pdf /tmp/preview/page
```

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Fonts display wrong on recipient's machine | Embed fonts in ReportLab: `pdfmetrics.registerFont(TTFont(...))` |
| Tables split mid-row across pages | Use ReportLab `KeepTogether` or `splitFirst`/`splitLast` |
| Images blurry | Use original resolution; don't downscale before embedding |
| Cover CSS renders differently | Test with same Chrome version used for production |
| Chinese/Japanese text missing | Register CJK fonts (SimSun, MS Gothic) with ReportLab |
| Large file size | Compress images before embedding; use JPEG for photos, PNG for diagrams |

---

## Bundled Resources

### Scripts

| Script | Purpose |
|--------|---------|
| `scripts/palette.py` | Generate design tokens (colors, fonts) for a document type |
| `scripts/cover.py` | Generate HTML cover page from design tokens |
| `scripts/render_cover.js` | Render HTML cover to PDF via Chrome/Playwright |
| `scripts/render_body.py` | Generate body PDF via ReportLab from content blocks |
| `scripts/merge.py` | Merge cover.pdf + body.pdf into final.pdf |
| `scripts/fill_inspect.py` | Inspect PDF form field names and types |
| `scripts/fill_write.py` | Fill PDF form fields programmatically |
| `scripts/reformat_parse.py` | Extract structure from existing PDF for reformatting |
| `scripts/make.sh` | End-to-end pipeline: palette → cover → body → merge |
| `scripts/md2pdf.py` | Convert Markdown to typeset PDF via ReportLab (10 themes, CJK, TOC, bookmarks, zero browser deps) |

### Design System

`design/design.md` — Complete design system reference: 15 document type recipes, accent color palettes, font pairings, cover patterns, content block JSON schemas.

---

## Dependencies

- **Generation**: Python `reportlab`, `pypdf`
- **Table extraction**: Python `pdfplumber`
- **Cover rendering** (optional): Chrome (via CDP) or Playwright for HTML→PDF
- **Preview**: Poppler `pdftoppm` for PNG rendering
- **CJK fonts**: `reportlab[cjk]` or manual font registration
- **Bundled scripts**: Python 3.9+, Node.js 18+ (for cover rendering)

---

## Attribution

This skill incorporates knowledge from:
- [MiniMax Office Skills](https://github.com/MiniMax-AI/skills) (MIT) — Dual-engine architecture (HTML cover + ReportLab body), document type catalog, design system, content block types
- [lovstudio/md2pdf](https://github.com/lovstudio/md2pdf) (MIT) — Pure ReportLab Markdown→PDF engine, CJK dual-layer mixing, 10 design themes, full document structure
