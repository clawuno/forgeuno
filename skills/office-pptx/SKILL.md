---
name: office-pptx
display_name: Office PPTX
version: "2.0.0"
description: >-
  Create, edit, inspect, and verify PowerPoint presentations (.pptx) as coherent
  visual narratives. Uses PptxGenJS for new decks, XML editing to preserve
  existing templates, and rendered contact-sheet plus full-slide review before
  delivery. Use for PowerPoint, PPTX, slides, presentations, decks, and pitch decks.
tags:
  - document
  - office
  - powerpoint
  - presentation
  - slides
allowed-tools: >-
  Read Write Edit Glob Grep Bash
---

# Office PPTX

Create presentations that make an argument at presentation distance. A deck is
not a document split into rectangles and not a sequence of interchangeable card
grids. Content, narrative, and visual direction determine the slide system.

## Task Routing

| Task | Route | Method |
|------|-------|--------|
| Create a new presentation | **CREATE** | PptxGenJS with reproducible source |
| Edit an existing presentation | **EDIT** | Preserve its design system; prefer XML edits for template fidelity |
| Read or extract content | **READ** | markitdown or python-pptx |

For CREATE or a visual redesign, use Clawuno's `company/_system-visual-design`
guidance when available. User requirements, supplied templates, brand assets,
and Workspace `DESIGN.md` take priority over defaults in this skill.

## CREATE Workflow

### 1. Establish the communication job

Identify the audience, setting, decision or change in understanding, time
available, source material, required format, language, and delivery constraints.
Infer reversible choices; ask only when a missing constraint would materially
change the deck.

### 2. Build a narrative spine

Outline the argument before slide layouts. Give each slide one communicative job
and a working assertion title where the evidence supports a conclusion. Sequence
slides so each creates the need for the next. Use agenda, section, appendix, and
summary slides only when they help this audience.

Read [slide roles and layout families](references/slide-types.md) when planning
the sequence. The roles are a vocabulary, not a fixed template or mandatory list.

### 3. Define a specific visual direction

Write one internal direction sentence that guides hierarchy, density, type,
color roles, imagery, chart treatment, and recurring motifs. Avoid generic
phrases such as "modern, clean, professional" and avoid choosing a style from a
fixed palette menu without relating it to the content.

Read [design-system guidance](references/design-system.md) for visual grammar,
typography, color, imagery, CJK, and consistency rules.

### 4. Plan visual rhythm

Create a slide plan containing: slide job, key message, evidence, visual form,
and layout family. Establish recurring anchors, then vary composition with the
story. Alternate evidence-dense moments with spacious synthesis or transition
moments. Do not repeat the same layout mechanically.

### 5. Author reproducibly

- Default to 16:9 (`LAYOUT_WIDE`) unless the source or user requires otherwise.
- Build with PptxGenJS and retain the source `.js` beside the `.pptx`.
- Centralize theme tokens, geometry, typography roles, and reusable helpers.
- Use `autoFontSize` or `calcTextBox`; do not trust built-in `fit`/`autoFit` for
  content whose rendered size matters.
- Use `imageSizingCrop` or `imageSizingContain`; never stretch an image.
- Prefer editable native charts for common chart types. Use SVG/PNG only when
  the required visual cannot be expressed reliably as an editable chart.
- Use bullet options rather than literal bullet characters.
- Render equations to SVG. Use a consistent monospace treatment for code.

Read [PptxGenJS reference](references/pptxgenjs.md) before implementation and
[pitfalls](references/pitfalls.md) before final generation.

### 6. Render and review

Render the complete deck to PDF and slide images. Review in two passes:

1. **Contact sheet**: narrative flow, rhythm, repeated layouts, palette, density,
   and whether the deck feels like one designed system.
2. **Full-size slides**: typography, wrapping, crop, alignment, chart labels,
   source notes, glyphs, and visual artifacts.

Extract text as a separate content check. Visual review and text extraction are
complementary; neither replaces the other. Fix material issues and re-render the
affected slides. See [visual review](references/visual-review.md).

### 7. Validate and deliver

Source must run overlap and bounds checks:

```javascript
warnIfSlideHasOverlaps(slide, pptx)
warnIfSlideElementsOutOfBounds(slide, pptx)
```

Resolve all unintentional overlaps and out-of-bounds elements. Deliver the
`.pptx`, reproducible source, and any required assets. Do not deliver temporary
preview images unless the user asks for them.

## EDIT Route

Inspect the existing deck before changing it. Treat its masters, layouts,
geometry, typography, recurring elements, imagery, and chart language as a
design system. Preserve it unless the user requests a redesign.

For template-preserving edits, use unpack -> edit XML -> repack:

```bash
mkdir /tmp/pptx_work && cd /tmp/pptx_work
unzip -o input.pptx -d unpacked/
# Edit the smallest necessary XML or media files.
cd unpacked && zip -r ../output.pptx . -x ".*"
```

Use XML editing when PptxGenJS would discard masters, layouts, animations,
relationships, embedded media, or template-specific behavior. Read
[editing existing presentations](references/editing.md) before changing package
internals. Render and compare the edited deck before delivery.

## READ Route

```bash
# Text extraction
python -m markitdown presentation.pptx

# Structured inspection
python3 - <<'PY'
from pptx import Presentation
prs = Presentation('presentation.pptx')
for i, slide in enumerate(prs.slides, start=1):
    print(f'--- Slide {i} ---')
    for shape in slide.shapes:
        if getattr(shape, 'has_text_frame', False):
            print(shape.text)
PY
```

Reading text does not establish visual quality. Render when layout, charts,
images, typography, or template fidelity matters.

## Non-Negotiable Quality Rules

- One primary message or communicative job per slide.
- Hierarchy must be visible at a glance and at presentation distance.
- Paragraphs and long lists are left-aligned; projection copy is edited down.
- Data charts use honest scales, units, sources, and a visual emphasis that serves
  the question rather than decoration.
- Images preserve aspect ratio, have adequate resolution, and use coherent crops.
- CJK and mixed-language text is checked in the rendered output for glyph support,
  line breaks, visual weight, punctuation, and font substitution.
- Placeholder content, hidden overflow, accidental collisions, and fake citations
  are release-blocking defects.
- Do not claim visual validation without reviewing rendered output.

## References

| Reference | Read when |
|-----------|-----------|
| [Slide roles and layout families](references/slide-types.md) | Planning narrative, slide jobs, and layout rhythm |
| [Design system](references/design-system.md) | Choosing type, color, imagery, geometry, and CJK behavior |
| [PptxGenJS](references/pptxgenjs.md) | Creating a deck programmatically |
| [Editing](references/editing.md) | Preserving an existing template or package structure |
| [Pitfalls](references/pitfalls.md) | Implementing and debugging PptxGenJS or OOXML |
| [Visual review](references/visual-review.md) | Rendering, contact-sheet review, and release checks |

## Dependencies

- Creation: Node.js + `pptxgenjs`
- Editing: `unzip`, `zip`
- Reading: `python-pptx` or `markitdown`
- Preview: LibreOffice (`soffice`) + Poppler (`pdftoppm`)
- Optional image processing: `sharp`

## Attribution

This Forgeuno skill is an original synthesis informed by:

- MiniMax Office Skills (MIT): theme and OOXML editing workflows
- OpenAI Skills (Apache-2.0): PptxGenJS helpers and render-and-verify discipline
- OpenDesign (Apache-2.0): separation of design direction, design systems, craft,
  artifact skills, and multi-lens visual critique
