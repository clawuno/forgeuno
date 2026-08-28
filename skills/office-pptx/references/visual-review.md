# Rendered Visual Review

## Render

Use the installed LibreOffice and Poppler tools, or the platform's bundled
equivalent. Keep intermediate files outside the final delivery directory.

```bash
mkdir -p /tmp/pptx-review
soffice --headless --convert-to pdf --outdir /tmp/pptx-review output.pptx
pdftoppm -png -r 150 /tmp/pptx-review/output.pdf /tmp/pptx-review/slide
```

Create a contact sheet with an available image tool when there are multiple
slides. If no contact-sheet helper exists, inspect the rendered PNGs in sequence.

## Pass 1: Contact Sheet

Review the deck as a whole:

- Does the sequence communicate a coherent argument?
- Is there one recognizable design direction?
- Do density and scale create rhythm, or is every slide equally busy?
- Is one layout repeated mechanically?
- Are opening, evidence, transition, and close moments appropriately distinct?
- Does color guide attention consistently?

## Pass 2: Full Size

Inspect representative slides and every visually risky slide:

- titles and body text wrap intentionally;
- no clipping, overlap, hidden overflow, or off-canvas content;
- fonts and CJK glyphs render as intended;
- image crops preserve subjects and aspect ratio;
- charts have legible labels, honest axes, units, and sources;
- lines, icons, and shapes have consistent visual weight;
- fine alignment and spacing are deliberate;
- footnotes and citations remain readable.

## Content and Structure Pass

Extract text separately:

```bash
python -m markitdown output.pptx > /tmp/pptx-review/content.txt
grep -iE "lorem|ipsum|placeholder|xxxx" /tmp/pptx-review/content.txt
```

Check slide order, missing content, duplicate content, spelling, and placeholders.
Run overlap and bounds checks from the source. These checks do not replace visual
inspection because they cannot judge hierarchy, crop, rhythm, or font substitution.

## Fix and Re-render

List material issues, fix them, and re-render the affected output. Stop when the
remaining differences are subjective or another pass would not materially improve
the audience outcome. Do not require a cosmetic change merely to claim an iteration.

If rendering is unavailable, disclose that limitation and report only structural
or source validation. Never describe an unobserved deck as visually verified.
