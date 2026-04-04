---
name: office-pptx
display_name: Office PPTX
version: "1.0.1"
description: >-
  Create, edit, and format PowerPoint presentations (.pptx) with consistent
  visual design. Defines 5 slide types, 4 style recipes, theme contract,
  and validation pipeline. Uses PptxGenJS for creation and XML editing
  for preserving existing templates.
  Keywords: PowerPoint, pptx, slides, presentation, deck, PptxGenJS.
tags:
  - document
  - office
  - powerpoint
  - presentation
  - slides
allowed-tools: >-
  Read Write Edit Bash
---

# Office PPTX — Presentation Design with Visual Consistency

Create presentations that look professionally designed — consistent typography,
spacing, colors, and visual hierarchy across every slide.

---

## Task Routing

| Task | Route | Method |
|------|-------|--------|
| Create new presentation from scratch | **CREATE** | PptxGenJS (JavaScript) |
| Edit existing .pptx | **EDIT** | XML unpack/edit/repack (preserves formatting) |
| Read/extract content from .pptx | **READ** | markitdown or python-pptx |

---

## Route: CREATE

### Workflow (7 Steps)

1. **Define theme** — Choose style recipe, color palette, fonts
2. **Plan slide structure** — Assign slide types to content sections
3. **Set slide size** — Default 16:9 (`LAYOUT_WIDE`). Match source material if recreating
4. **Build slides** — Per-slide JS modules using PptxGenJS
5. **Render preview** — Convert to PDF/PNG via LibreOffice
6. **Validate** — Check overflow, font substitution, visual consistency
7. **Deliver** — Output .pptx + source .js for reproducibility

### 5 Slide Types

Every slide must be one of these types:

| Type | Purpose | Layout |
|------|---------|--------|
| **Cover** | Title slide, first impression | Full-bleed background, large title, subtitle, date/author |
| **TOC** | Table of contents, navigation | Numbered section list, optional icons |
| **Section Divider** | Chapter/topic transition | Large section number + title, accent background |
| **Content** | Main information delivery | Title + body area (text, images, charts, tables) |
| **Summary** | Conclusion, key takeaways | Bullet points or numbered recap, CTA |

### 4 Style Recipes

Each recipe defines a complete visual system:

| Recipe | Border Radius | Shadow | Spacing | Personality |
|--------|--------------|--------|---------|-------------|
| **Sharp** | 0px | Hard, offset | Tight | Corporate, formal, editorial |
| **Soft** | 4-8px | Soft, blurred | Balanced | Professional, modern, clean |
| **Rounded** | 12-16px | Medium | Generous | Friendly, approachable, startup |
| **Pill** | Full radius | Subtle | Spacious | Playful, creative, consumer |

### Theme Object Contract

Every presentation must define a theme object with these 5 keys:

```javascript
const theme = {
  colors: {
    primary: '#1B365D',     // Main brand color
    secondary: '#4A90D9',   // Accent, highlights
    background: '#FFFFFF',  // Slide background
    text: '#333333',        // Body text
    muted: '#999999',       // Captions, metadata
  },
  fonts: {
    heading: 'Montserrat',  // Bold, display
    body: 'Open Sans',      // Regular, readable
    code: 'Fira Code',      // Monospace (if needed)
  },
  recipe: 'Soft',           // Sharp | Soft | Rounded | Pill
  slideSize: 'LAYOUT_WIDE', // 16:9
  pageNumberBadge: true,     // Show page numbers on all non-cover slides
}
```

### Color Palette Reference

| Use Case | Palette | Colors |
|----------|---------|--------|
| Corporate | Navy + Gold | `#1B365D` `#D4AF37` `#F5F5F5` |
| Technology | Blue + Cyan | `#0066FF` `#00D4FF` `#F0F8FF` |
| Creative | Coral + Purple | `#FF6B6B` `#9B59B6` `#FFF5F5` |
| Nature | Green + Brown | `#228B22` `#8B4513` `#F0FFF0` |
| Minimal | Gray + Black | `#333333` `#666666` `#FAFAFA` |
| Bold | Red + Black | `#E74C3C` `#2C3E50` `#FFFFFF` |

---

## Route: EDIT (Existing Templates)

**Same principle as XLSX: unpack → edit XML → repack.**

```bash
# 1. Unpack
mkdir /tmp/pptx_work && cd /tmp/pptx_work
unzip -o input.pptx -d unpacked/

# 2. Edit slide XML (e.g., ppt/slides/slide1.xml)
# Find and replace placeholder text, update values, modify elements

# 3. Repack
cd unpacked && zip -r ../output.pptx . -x ".*"
```

**Why XML editing for existing files?** PptxGenJS creates new files — it can't preserve an existing template's master slides, layouts, animations, and embedded media. XML editing preserves everything.

### Common Edit Operations

- **Replace placeholder text**: Find `<a:t>PLACEHOLDER</a:t>` → replace inner text
- **Update chart data**: Edit `ppt/charts/chart1.xml` data references
- **Change image**: Replace file in `ppt/media/`, keep same filename
- **Add slide**: Copy an existing slide XML, add to `[Content_Types].xml` and relationships

---

## Route: READ

```bash
# markitdown (text extraction)
markitdown presentation.pptx

# python-pptx (structured access)
python3 -c "
from pptx import Presentation
prs = Presentation('file.pptx')
for i, slide in enumerate(prs.slides):
    print(f'--- Slide {i+1} ---')
    for shape in slide.shapes:
        if shape.has_text_frame:
            print(shape.text)
"
```

---

## PptxGenJS Authoring Rules

### Text Sizing

- Use `autoFontSize` or `calcTextBox` helpers for text boxes — do not use PptxGenJS built-in `fit` or `autoFit` (unreliable)
- Minimum readable font: 14pt for body, 24pt for titles, 10pt for footnotes

### Images

- Use `imageSizingCrop` or `imageSizingContain` — not PptxGenJS built-in image sizing
- Convert SVG/EMF/HEIC to PNG before embedding (broad compatibility)
- Maintain aspect ratio; never stretch

### Lists

- Use PptxGenJS bullet options, not literal `•` characters
- Consistent indent level: 0.5 inch per level

### Charts

- Prefer native PowerPoint charts for bar/line/pie (editable by recipients)
- For complex charts PptxGenJS can't express, render as SVG/PNG and embed as image

### Equations and Code

- Equations: Render LaTeX → SVG → embed as image
- Code blocks: Use monospace font with syntax highlighting colors

### Validation

Include these checks in the source .js:

```javascript
warnIfSlideHasOverlaps(slide, pptx)
warnIfSlideElementsOutOfBounds(slide, pptx)
```

Fix all unintentional overlaps and out-of-bounds before delivering.

---

## Validation Pipeline

### Automated Checks

```bash
# Render slides to PNG
soffice --headless --convert-to pdf input.pptx
pdftoppm -png input.pdf /tmp/slides/page

# Check for overflow (if using PptxGenJS test script)
node slides_test.js input.pptx

# Detect font substitution
# Compare intended fonts with what LibreOffice actually renders
```

### Visual Checklist

- [ ] All slides use consistent fonts (no system fallbacks)
- [ ] Colors match the theme palette (no stray colors)
- [ ] Text doesn't overflow text boxes
- [ ] Images are sharp and properly sized
- [ ] Page numbers appear on all non-cover slides
- [ ] Slide transitions are consistent (or none)
- [ ] Master slide/layout is applied correctly
- [ ] Spacing between elements is consistent across slides

### CJK Fonts

For Chinese/Japanese/Korean content:
- Use Microsoft YaHei (Chinese), Meiryo (Japanese), Malgun Gothic (Korean)
- Verify fonts render correctly in LibreOffice preview

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Fonts render differently on recipient's machine | Use widely available fonts or embed; test with LibreOffice |
| Text overflows box | Use `autoFontSize` helper or reduce text; never hide overflow |
| Slides look inconsistent | Enforce theme object contract — every slide uses same colors/fonts/recipe |
| Edit destroys template animations | Use XML editing, not PptxGenJS for existing files |
| Chart not editable | Use native PowerPoint chart types, not embedded images |
| Large file size | Compress images before embedding; remove unused master slides |

---

## Bundled Resources

### References

| Reference | Content |
|-----------|---------|
| `references/slide_types.md` | Detailed specs for 5 slide types (Cover, TOC, Section, Content, Summary) with layout rules |
| `references/design_system.md` | Color palettes, font reference, style recipe parameters, spacing rules |
| `references/editing.md` | XML-level editing guide for existing .pptx templates |
| `references/pitfalls.md` | Common issues and QA process |
| `references/pptxgenjs_api.md` | PptxGenJS API quick reference and helper usage |

---

## Dependencies

- **Creation**: Node.js + `pptxgenjs` (JavaScript)
- **Editing**: `unzip`, `zip` (for XML-level editing)
- **Reading**: Python `python-pptx` or `markitdown`
- **Preview**: LibreOffice (`soffice`) + Poppler (`pdftoppm`)
- **Helpers** (optional): `sharp` for image processing, `react-icons` for icon SVGs

---

## Attribution

This skill incorporates knowledge from:
- [MiniMax Office Skills](https://github.com/MiniMax-AI/skills) (MIT) — Theme contract, 5 slide types, style recipes, color palette system, XML editing workflow
- [OpenAI Skills](https://github.com/openai/skills) (Apache-2.0) — PptxGenJS helper utilities (autoFontSize, calcTextBox, overflow/bounds detection), validation scripts
