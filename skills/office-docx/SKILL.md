---
name: office-docx
display_name: Office DOCX
version: "1.0.0"
description: >-
  Create, edit, and format Word documents (.docx) with production-grade quality.
  Covers three pipelines: create from scratch, edit existing documents preserving
  structure, and apply templates. Includes OpenXML knowledge for element ordering,
  font units, CJK typography, track changes, and validation.
  Keywords: Word, docx, document, OpenXML, template, formatting, report, proposal.
tags:
  - document
  - office
  - word
  - docx
  - openxml
allowed-tools: >-
  Read Write Edit Bash
---

# Office DOCX — Production-Grade Word Document Processing

Create, edit, and format Word documents that are genuinely deliverable — not just
"opens without error" but structurally correct, visually polished, and editable
by recipients.

---

## Pipeline Routing

Determine which pipeline to use based on the task:

| Task | Pipeline | Description |
|------|----------|-------------|
| Write a report/proposal/manual from scratch | **A: CREATE** | Generate complete .docx from content |
| Fill in or modify an existing .docx | **B: EDIT** | Preserve existing structure, change only target content |
| Apply a template to content or another .docx | **C: FORMAT** | Overlay or replace styles from a template |

---

## Technology Stack

**.NET OpenXML SDK** — the only recommended library for all platforms (macOS, Linux, Windows).

OpenXML SDK is Microsoft's official implementation of the ECMA-376 standard, providing complete and reliable control over Word document structure. It ships as a self-contained executable (~30MB) bundled with the Clawuno installation — no separate .NET install needed.

**LibreOffice CLI** — used for .doc→.docx conversion and visual preview rendering (PDF/PNG). Not for document generation.

---

## Pipeline A: CREATE from Scratch

### Workflow

1. **Plan structure** — Outline sections, heading levels, content blocks
2. **Generate document** — Create .docx with proper styles, not inline formatting
3. **Validate** — Check element ordering, run merge, XSD-level validation
4. **Visual review** — Render to PDF/PNG via LibreOffice, inspect every page

### Critical Rules for Creation

**Use styles, not direct formatting.** Every heading must use a named style (Heading 1, Heading 2...) with `outlineLevel` set — otherwise Table of Contents won't work.

**Element ordering matters.** OpenXML requires child elements in a specific order. Wrong order = corrupt file. Key orderings:

```
w:body children:    w:p | w:tbl | w:sdt (in sequence)
w:p children:       w:pPr → w:r (properties before runs)
w:r children:       w:rPr → w:t | w:br | w:drawing (properties before content)
w:tbl children:     w:tblPr → w:tblGrid → w:tr
w:tc children:      w:tcPr → w:p (every cell must end with w:p)
```

**Font size units.** OpenXML uses half-points: `<w:sz w:val="24"/>` = 12pt. Always multiply desired pt × 2.

**Spacing units.** Line spacing uses 240ths of a line. `<w:spacing w:line="360"/>` = 1.5 line spacing (360/240 = 1.5).

### Content Block Patterns

| Block | Implementation |
|-------|---------------|
| **Heading** | Named style with outlineLevel. Never use bold paragraph as fake heading |
| **Table** | `w:tblGrid` defines columns. Every `w:tc` must contain at least one `w:p` |
| **Image** | Inline drawing with EMU units (1 inch = 914400 EMU) |
| **Page break** | `<w:br w:type="page"/>` inside a run, not a separate paragraph |
| **Header/Footer** | Per-section via `w:sectPr` → `w:headerReference` |
| **TOC** | `w:sdt` with `HYPERLINK \l` fields. Requires heading styles with outlineLevel |
| **Footnote** | `w:footnoteReference` in run + `w:footnote` in footnotes part |

---

## Pipeline B: EDIT Existing Document

### Workflow

1. **Read and analyze** — Parse structure, identify target sections
2. **Edit surgically** — Modify only the necessary elements
3. **Preserve everything else** — Styles, formatting, track changes, headers, macros
4. **Validate** — Verify no structural damage after edit
5. **Visual review** — Compare before/after rendering

### Critical Rules for Editing

**Never recreate the document.** Open, modify, save — do not extract content and rebuild. Rebuilding destroys:
- Custom styles and theme
- Track changes history
- Header/footer per-section bindings
- Embedded objects and relationships

**Prevent direct format contamination.** When copying content between documents, strip `w:rPr` (run properties) and `w:pPr` (paragraph properties) that carry source-document formatting. Apply target document's styles instead.

**python-docx specific:** When editing, always open with `Document(path)` not `Document()`. The latter creates a blank document.

**OpenXML SDK specific:** Use `WordprocessingDocument.Open(path, isEditable: true)`. Dispose properly to flush changes.

---

## Pipeline C: FORMAT / Template Application

### Two Approaches

**C-1: Overlay** — Keep document content, replace styles from template
- Copy `styles.xml` from template to target
- Update `w:defaultStyles` and named styles
- Preserve content paragraphs and runs

**C-2: Base Replace** — Use template as base, inject content
- Open template as new document
- Find and replace placeholder content
- Maintain template's section properties, headers, footers

### Multi-Template Merge (Advanced)

For documents needing different templates per section (e.g., thesis with different chapter styles):
1. Generate each section as separate .docx with its template
2. Merge using section breaks (`w:sectPr` with `w:type="nextPage"`)
3. Each section retains its own header/footer bindings

---

## CJK Typography (Chinese/Japanese/Korean)

When document contains CJK text:

| Rule | Implementation |
|------|---------------|
| **Font stack** | East Asian: SimSun/MS Mincho/Malgun Gothic. Latin: Times New Roman |
| **Paragraph spacing** | Use `w:snapToGrid` for line grid alignment |
| **Character spacing** | `w:spacing w:val="0"` (no extra kerning for CJK) |
| **Punctuation** | Enable `w:kinsoku` for line-break rules |
| **Chinese gov docs (GB/T 9704)** | A4, 3.7cm top/bottom, 2.8cm left/right, SimHei for titles, FangSong for body |

---

## Validation Checklist

Before delivering any .docx:

- [ ] File opens without repair prompt in Word/LibreOffice
- [ ] All headings use named styles (not bold/large font)
- [ ] Table of Contents updates correctly (if present)
- [ ] Headers/footers display correctly per section
- [ ] Images render at correct size and position
- [ ] Track changes (if any) are preserved
- [ ] Fonts are available or embedded
- [ ] Page numbers are sequential
- [ ] No orphan/widow paragraphs at page breaks

### LibreOffice Preview Command

```bash
# DOCX → PDF
soffice --headless --convert-to pdf --outdir /tmp/preview input.docx

# PDF → PNG pages
pdftoppm -png /tmp/preview/input.pdf /tmp/preview/page
```

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| TOC shows "Error! No table of contents entries found" | Headings must use named styles with outlineLevel, not just bold text |
| File opens with "repair" dialog | Element ordering is wrong — check w:body, w:p, w:r child order |
| Formatting lost after template merge | Direct format contamination — strip w:rPr/w:pPr from source before merge |
| Headers different on first page but shouldn't be | Check `w:titlePg` in section properties |
| Font displays wrong on recipient's machine | Embed fonts or use universally available fonts (Calibri, Arial, Times New Roman) |
| Image appears as empty box | Check relationship ID matches, EMU dimensions are non-zero |

---

## Bundled Resources

This skill includes scripts, references, assets, and C# source code available at `https://forgeuno.com/skills/office-docx/`. Download what you need to the workspace before use.

### Scripts

| Script | Purpose |
|--------|---------|
| `scripts/setup.sh` / `setup.ps1` | Install .NET SDK and OpenXML dependencies |
| `scripts/env_check.sh` | Verify .NET and OpenXML SDK are available |
| `scripts/doc_to_docx.sh` | Convert .doc to .docx via LibreOffice |
| `scripts/docx_preview.sh` | Render .docx to PDF/PNG for visual review |

### .NET CLI Project

`scripts/dotnet/` contains a complete .NET solution:
- **MiniMaxAIDocx.Cli** — CLI entry point for document operations
- **MiniMaxAIDocx.Core/Commands/** — Create, Edit, ApplyTemplate, Validate, Analyze, Diff, FixOrder, MergeRuns
- **MiniMaxAIDocx.Core/OpenXml/** — ElementOrder, RunMerger, StyleAnalyzer, UnitConverter, TrackChangesHelper
- **MiniMaxAIDocx.Core/Samples/** — 16 compilable C# samples covering all OpenXML topics (tables, images, lists, styles, headers, footnotes, track changes, 13 aesthetic recipes)
- **MiniMaxAIDocx.Core/Validation/** — XSD validator, business rules validator, gate check

Usage: `dotnet run --project scripts/dotnet/MiniMaxAIDocx.Cli -- <command> [args]`

### References (18 docs)

| Reference | Content |
|-----------|---------|
| `references/scenario_a.md` | Guide for CREATE pipeline |
| `references/scenario_b.md` | Guide for EDIT pipeline |
| `references/scenario_c.md` | Guide for FORMAT/template pipeline |
| `references/openxml_encyclopedia_part1.md` | OpenXML elements reference (paragraphs, runs, styles) |
| `references/openxml_encyclopedia_part2.md` | OpenXML elements reference (tables, images, fields) |
| `references/openxml_encyclopedia_part3.md` | OpenXML elements reference (sections, headers, TOC) |
| `references/cjk_typography.md` | Chinese/Japanese/Korean typography standards |
| `references/typography_guide.md` | General typography and page layout |
| `references/design_principles.md` | Document design principles and aesthetic recipes |
| `references/troubleshooting.md` | Common errors and fixes |
| Other references | Additional format-specific guides |

### Assets

| Asset | Purpose |
|-------|---------|
| `assets/styles/` | Pre-built style XML files (academic, corporate, default) |
| `assets/xsd/` | XSD schemas for document validation (aesthetic rules, business rules, common types, WML subset) |

### How to Use

1. Set up .NET environment: `bash scripts/setup.sh`
2. Verify: `bash scripts/env_check.sh`
3. Create document: `dotnet run --project scripts/dotnet/MiniMaxAIDocx.Cli -- create --output report.docx`
4. Load references as needed: `curl -s https://forgeuno.com/skills/office-docx/references/scenario_a.md`

---

## Dependencies

- **.NET OpenXML SDK**: Bundled with Clawuno as self-contained CLI (all platforms, ~30MB). Or install via `scripts/setup.sh`
- **Preview**: LibreOffice (`soffice`) + Poppler (`pdftoppm`) for visual verification

---

## Attribution

This skill incorporates knowledge from:
- [MiniMax Office Skills](https://github.com/MiniMax-AI/skills) (MIT) — OpenXML element ordering, font unit math, CJK typography standards, template contamination prevention, XSD validation pipeline
- [OpenAI Skills](https://github.com/openai/skills) (Apache-2.0) — Visual render-and-inspect verification workflow
