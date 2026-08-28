# QA Process & Common Pitfalls

## QA Process

Approach QA as a defect search rather than a confirmation ritual. Inspect the
render critically, but do not invent a cosmetic change merely to prove that an
iteration happened.

### Content QA

```bash
python -m markitdown output.pptx
```

Check for missing content, typos, wrong order.

**Check for leftover placeholder text:**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|placeholder|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

### Verification Loop

1. Generate slides -> Extract text with `python -m markitdown output.pptx` -> Review content
2. List material issues found
3. Fix issues that affect communication, fidelity, or delivery
4. **Re-verify affected slides** - one fix can create another problem
5. Repeat until a full pass reveals no new issues

### Per-Slide QA (for from-scratch creation)

```bash
python -m markitdown slide-XX-preview.pptx
```

Check for missing content and placeholder text. Verify folios only when the deck's
design system or user request includes them.

---

## Common Mistakes to Avoid

- **Don't repeat the same layout mechanically** - vary composition when the content relationship changes
- **Don't center body text** - left-align paragraphs and lists; center only short display statements
- **Don't solve overflow with tiny type** - edit or split content before shrinking below a readable presentation size
- **Don't default to a fashionable palette** - derive color roles from the topic, audience, and brand
- **Don't mix spacing randomly** - define a small spacing rhythm and use it consistently
- **Don't style one slide in isolation** - treat the deck as one visual system
- **Don't add decorative visuals to satisfy a quota** - text-led slides are valid when typography and space carry the idea
- **Don't forget text box padding** - when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
- **Don't use low-contrast elements** - icons and text need strong contrast against the background
- **Don't add formulaic title decoration** - use a rule or accent only when it belongs to the chosen visual language
- **NEVER use "#" with hex colors** — causes file corruption in PptxGenJS
- **NEVER encode opacity in hex strings** — use the `opacity` property instead
- **NEVER use async/await in createSlide()** — compile.js won't await
- **NEVER reuse option objects across PptxGenJS calls** — PptxGenJS mutates objects in-place

---

## Critical Pitfalls — PptxGenJS

### NEVER use async/await in createSlide()

```javascript
// WRONG - compile.js won't await
async function createSlide(pres, theme) { ... }

// CORRECT
function createSlide(pres, theme) { ... }
```

### NEVER use "#" with hex colors

```javascript
color: "FF0000"      // CORRECT
color: "#FF0000"     // CORRUPTS FILE
```

### NEVER encode opacity in hex strings

```javascript
shadow: { color: "00000020" }              // CORRUPTS FILE
shadow: { color: "000000", opacity: 0.12 } // CORRECT
```

### Prevent unintended title wrapping

Measure with `calcTextBox` or an `autoFontSize` helper, then edit the title or
allocate sufficient width and height. Do not depend on built-in `fit` behavior
for a title whose final line break matters.

### NEVER reuse option objects across calls

```javascript
// WRONG
const shadow = { type: "outer", blur: 6, offset: 2, color: "000000", opacity: 0.15 };
slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });
slide.addShape(pres.shapes.RECTANGLE, { shadow, ... });

// CORRECT - factory function
const makeShadow = () => ({ type: "outer", blur: 6, offset: 2, color: "000000", opacity: 0.15 });
slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });
slide.addShape(pres.shapes.RECTANGLE, { shadow: makeShadow(), ... });
```
