# Presentation Design System

Build a small, explicit visual system from the deck's communication job. Do not
select a canned palette and corner-radius recipe before understanding the content.

## Direction to Tokens

Translate the internal visual direction into reusable decisions:

- **Canvas**: light, dark, paper-like, photographic, or another intentional field
- **Type roles**: display, slide title, body, label, source, numeric, and code
- **Color roles**: canvas, surface, primary/secondary text, accent, semantic, data
- **Geometry**: margins, columns, baseline rhythm, corner language, line weights
- **Imagery**: photography, illustration, diagram, screenshot, texture, or none
- **Motif**: at most one or two recurring devices tied to the story

Centralize these values in the source. Do not scatter magic numbers and colors
through slide modules.

```javascript
const theme = {
  colors: {
    canvas: 'F7F4EE',
    surface: 'FFFFFF',
    text: '172026',
    muted: '5F6B72',
    accent: 'B33A2B',
    line: 'D8D2C8',
  },
  fonts: {
    display: 'Aptos Display',
    body: 'Aptos',
    cjk: 'Microsoft YaHei',
    code: 'Consolas',
  },
  layout: {
    marginX: 0.55,
    marginTop: 0.45,
    gap: 0.28,
  },
}
```

The example demonstrates roles, not a universal palette.

## Typography

- Make the title/body distinction unmistakable at presentation distance.
- Use the fewest type families needed; create range through scale, weight, width,
  case, and spacing.
- Prefer assertion titles when a slide communicates a conclusion.
- Keep paragraphs and lists left-aligned. Center only short display statements.
- Avoid body text below 16pt when possible; 18-24pt is a safer presentation range.
  Sources may be smaller but must remain readable in the final render.
- Do not shrink large amounts of prose to make them fit. Edit or split the idea.

### CJK and Mixed Language

- Use a CJK face available in the delivery environment and define a fallback.
- Judge actual glyph weight; equal point sizes do not imply equal visual size.
- Allow comfortable line height and verify punctuation, numerals, and Latin runs.
- Avoid artificial tracking on CJK text.
- Render-test every font choice. Font substitution is a visual defect, not a minor
  implementation detail.

## Color

- Give every color a role. One main accent is often enough.
- Use contrast to direct attention, not to decorate every container.
- Keep data-series colors distinguishable and label important series directly.
- Do not rely on color alone for status or category.
- PptxGenJS colors omit `#`; encode transparency with the supported transparency
  or opacity property, never with 8-digit hex values.
- Gradients are optional, not forbidden. Use them only when they are integral to
  the direction and render consistently in the target software.

## Geometry and Rhythm

- Use shared alignment lines across slides even when compositions vary.
- Make proximity show relationships before adding borders or cards.
- Keep recurring elements in stable positions: titles, sources, folios, or section
  markers only when they are part of the system.
- Vary density with the narrative. A deck with identical density on every slide
  feels generated rather than presented.
- Corner radius, shadow, and line treatment must express the direction. Do not use
  rounded cards merely because they are easy to generate.

## Imagery and Icons

- Select images for narrative value and crop around the subject.
- Never stretch. Use crop or contain helpers and verify the result.
- Keep photography, illustration, screenshots, and icons stylistically coherent.
- Prefer a meaningful image, diagram, or chart over a row of decorative icons.
- Do not fabricate logos, screenshots, citations, or image rights.

## Charts

- Start with the audience question, then choose the chart.
- Show units, source, relevant baseline, and uncertainty.
- Highlight the evidence that supports the slide message; mute context.
- Avoid 3D charts, decorative gauges, and unnecessary legends.
- Prefer editable native charts when they can express the intended design reliably.

## Consistency vs Repetition

Consistency means shared rules. Repetition means copying the same layout. Keep
type roles, color roles, alignment logic, and motif coherent while allowing each
slide's content to choose an appropriate composition.
