---
name: office-xlsx
display_name: Office XLSX
version: "1.0.1"
description: >-
  Create, edit, and analyze Excel spreadsheets (.xlsx, .csv) with formula integrity
  and format preservation. Uses XML-level editing for existing files to prevent
  corruption of VBA, pivot tables, and sparklines. Covers financial formatting
  conventions, formula hygiene, and validation workflows.
  Keywords: Excel, xlsx, csv, spreadsheet, formula, pivot table, financial model.
tags:
  - document
  - office
  - excel
  - spreadsheet
  - financial
allowed-tools: >-
  Read Write Edit Bash
---

# Office XLSX — Spreadsheet Processing with Formula Integrity

Create and edit spreadsheets that preserve formulas, formatting, VBA macros, pivot
tables, and sparklines. The key principle: **never silently corrupt data during
read-write cycles.**

---

## Task Routing

| Task | Route | Method |
|------|-------|--------|
| Read data, analyze, aggregate | **READ** | openpyxl (read-only) or pandas |
| Create new spreadsheet from scratch | **CREATE** | XML template + manual XML editing |
| Edit existing spreadsheet | **EDIT** | XML unpack → edit → repack |
| Repair broken formulas | **FIX** | Treat as EDIT |
| Verify formulas and structure | **VALIDATE** | Static check + LibreOffice recalculation |

---

## Why XML-Level Editing?

**openpyxl round-trip silently destroys data.** When you open a .xlsx with openpyxl and save it, the following are quietly discarded:
- Pivot tables
- Sparklines
- VBA macros
- Conditional formatting (partially)
- Data validation (partially)
- Comments in some formats

This is unacceptable for editing real-world files. The solution: **edit at the XML level**.

A .xlsx file is a ZIP archive containing XML files:

```
[Content_Types].xml
_rels/.rels
xl/
  workbook.xml          ← sheet names, relationships
  sharedStrings.xml     ← text cell values
  styles.xml            ← number formats, fonts, fills
  worksheets/
    sheet1.xml          ← cell data, formulas, formatting refs
  _rels/workbook.xml.rels
```

### XML Edit Pattern

```bash
# 1. Unpack
mkdir /tmp/xlsx_work && cd /tmp/xlsx_work
unzip -o input.xlsx -d unpacked/

# 2. Edit target XML files (e.g., xl/worksheets/sheet1.xml)
# ... modify only the specific cells/rows needed ...

# 3. Repack
cd unpacked && zip -r ../output.xlsx . -x ".*"
```

**Only modify what you need.** Everything else stays untouched — styles, charts, macros, pivot tables all survive.

---

## Route: READ

Use openpyxl in **read-only mode** or pandas:

```python
# openpyxl (preserves formatting info)
from openpyxl import load_workbook
wb = load_workbook('file.xlsx', read_only=True, data_only=True)
ws = wb.active
for row in ws.iter_rows(values_only=True):
    print(row)

# pandas (best for analysis)
import pandas as pd
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')
```

**Formatting rules for READ output:**
- Preserve decimal places as shown in the spreadsheet
- Currency: include symbol and 2 decimal places
- Percentages: display as `XX.X%`, not `0.XXX`
- Aggregations (sum, average, count): compute from DataFrame, don't invent numbers

---

## Route: CREATE

**Always start from an XML template**, not from `openpyxl.Workbook()`:

1. Copy a minimal template directory (containing valid XML structure)
2. Edit `xl/worksheets/sheet1.xml` to add your data
3. Update `xl/sharedStrings.xml` if adding text values
4. Pack into .xlsx

### Cell XML Structure

```xml
<!-- Number cell -->
<c r="A1" s="0"><v>42</v></c>

<!-- Text cell (shared string index) -->
<c r="A2" s="0" t="s"><v>0</v></c>

<!-- Formula cell -->
<c r="A3" s="0"><f>SUM(A1:A2)</f></c>

<!-- Date cell (Excel serial number, formatted via style) -->
<c r="A4" s="1"><v>45678</v></c>
```

### Formula-First Rule

**Every derived value must be a real Excel formula**, not a hardcoded number.

Bad: `<c r="C10"><v>1250</v></c>` (dead number — can't update when inputs change)
Good: `<c r="C10"><f>SUM(C2:C9)</f></c>` (live formula — updates automatically)

---

## Route: EDIT

### Critical Integrity Rules

1. **Never create a new Workbook for edits.** Open → modify → save. Creating new loses everything.
2. **Same sheets rule.** Output must contain exactly the same sheets as input (same names, same order).
3. **Verify after save.** Open the output, check that unmodified cells are unchanged.

### Common Edit Operations

**Fill cells with values:**
```xml
<!-- In xl/worksheets/sheet1.xml, find the target <c r="B2"> and update -->
<c r="B2" s="0"><v>NEW_VALUE</v></c>
```

**Add a column:** Insert new `<c>` elements in each `<row>`, shift existing column references (A→B, B→C...). Update formulas that reference shifted columns.

**Insert a row:** Add new `<row>` element, increment `r` attributes of subsequent rows. Update formulas that reference shifted rows (e.g., `SUM(B2:B9)` becomes `SUM(B2:B10)` if row inserted within range).

**Row-wide borders:** Set style index on all cells in the row. Define border style in `xl/styles.xml`.

---

## Route: VALIDATE

### Static Validation

Check formulas without executing them:
- All `SUM`, `AVERAGE`, `COUNT` ranges reference existing cells
- No `#REF!`, `#NAME?`, `#VALUE!` in formula text
- Circular references: cell references itself directly or indirectly
- Off-by-one: `SUM(B2:B9)` when data extends to B10

### Dynamic Validation

Use LibreOffice to recalculate and compare:

```bash
# Recalculate formulas
soffice --headless --calc --convert-to xlsx --outdir /tmp input.xlsx

# Compare original cached values with recalculated values
# Discrepancy = formula or range error
```

---

## Financial Formatting Standards

When creating financial models or IB-style spreadsheets:

### Cell Color Conventions

| Color | Meaning | Hex |
|-------|---------|-----|
| Blue | User input / hardcoded values | `#0000FF` |
| Black | Formulas / derived values | `#000000` |
| Green | Cross-sheet references / linked values | `#00B050` |
| Gray | Static constants | `#808080` |
| Orange | Review / caution | `#FF8C00` |
| Red | Error / flag | `#FF0000` |

### Number Formatting

| Type | Format | Example |
|------|--------|---------|
| Currency | `$#,##0.00` | `$1,250.00` |
| Percentages | `0.0%` | `12.5%` |
| Multiples | `0.0"x"` | `5.2x` |
| Negative numbers | `#,##0.00;[Red](#,##0.00)` | Red in parentheses |
| Zero values | `#,##0.00;[Red](#,##0.00);"-"` | Dash for zero |
| Units in headers | — | `Revenue ($mm)` |

### IB Layout Standards

- Totals should SUM the range directly above (not cherry-pick cells)
- Hide gridlines; use horizontal borders above totals
- Section headers: merged cells with dark fill, white text
- Column labels: right-aligned for numbers, left-aligned for text
- Indent sub-metrics under parent line items

---

## Formula Hygiene

- **Use formulas for derived values**, never hardcode results
- **Avoid dynamic array functions**: `FILTER`, `XLOOKUP`, `SORT`, `SEQUENCE` (compatibility issues)
- **Keep formulas simple**: use helper cells for complex logic
- **Avoid volatile functions**: `INDIRECT`, `OFFSET` (unless required)
- **Use absolute references**: `$B$4` for constants, relative for copied formulas
- **Guard against errors**: wrap with `IFERROR` when appropriate
- **Never use `=TABLE`**: unsupported in most non-Excel environments

---

## Validation Checklist

Before delivering any .xlsx:

- [ ] File opens without errors in Excel/LibreOffice
- [ ] All formulas produce correct results (spot-check 5+ cells)
- [ ] No `#REF!`, `#DIV/0!`, `#VALUE!`, `#N/A`, `#NAME?` errors
- [ ] Formatting matches requirements (currency, percentages, dates)
- [ ] Unmodified content is unchanged (for EDIT tasks)
- [ ] Pivot tables / charts / macros still work (for EDIT tasks)
- [ ] Print layout is reasonable (page breaks, margins)

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Pivot tables disappear after save | Used openpyxl round-trip — use XML-level editing instead |
| Formulas show cached values, not recalculating | Open in Excel/LibreOffice to trigger recalculation |
| Shared strings corrupted | When adding text cells, update `xl/sharedStrings.xml` count attribute |
| Style applied to wrong cells | Style index in `<c s="X">` must match the correct index in `xl/styles.xml` |
| Formula range off-by-one after row insert | Must shift all formula references in affected rows |
| Date displays as number | Apply date number format via style index |

---

## Bundled Resources

### Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| `scripts/xlsx_unpack.py` | Unpack .xlsx into XML directory | `python3 xlsx_unpack.py input.xlsx /tmp/work/` |
| `scripts/xlsx_pack.py` | Repack XML directory into .xlsx | `python3 xlsx_pack.py /tmp/work/ output.xlsx` |
| `scripts/xlsx_reader.py` | Read cells, ranges, formatting info | `python3 xlsx_reader.py input.xlsx --range A1:D10` |
| `scripts/xlsx_add_column.py` | Insert column with data, shift existing | `python3 xlsx_add_column.py /tmp/work/ --col C --header "Total"` |
| `scripts/xlsx_insert_row.py` | Insert rows, shift existing | `python3 xlsx_insert_row.py /tmp/work/ --row 5 --count 3` |
| `scripts/xlsx_shift_rows.py` | Shift row references in formulas | `python3 xlsx_shift_rows.py /tmp/work/ --from 5 --offset 3` |
| `scripts/formula_check.py` | Static formula validation (ranges, errors) | `python3 formula_check.py /tmp/work/` |
| `scripts/libreoffice_recalc.py` | Dynamic validation via LibreOffice | `python3 libreoffice_recalc.py input.xlsx` |
| `scripts/shared_strings_builder.py` | Rebuild shared strings XML | `python3 shared_strings_builder.py /tmp/work/` |
| `scripts/style_audit.py` | Audit and report styles used | `python3 style_audit.py /tmp/work/` |

### References

| Reference | Content |
|-----------|---------|
| `references/create.md` | Detailed guide for creating spreadsheets from XML template |
| `references/edit.md` | Detailed guide for XML-level editing operations |
| `references/fix.md` | Guide for repairing broken formulas and structure |
| `references/format.md` | Financial formatting standards (34,000 words, IB-grade) |
| `references/validate.md` | Complete validation workflow and criteria |
| `references/read-analyze.md` | Guide for reading and analyzing existing spreadsheets |
| `references/ooxml-cheatsheet.md` | Quick reference for OOXML spreadsheet XML structure |

### Templates

`templates/minimal_xlsx/` — A valid minimal .xlsx structure (unpacked) for use as a CREATE starting point. Contains: `[Content_Types].xml`, `xl/workbook.xml`, `xl/worksheets/sheet1.xml`, `xl/styles.xml`, `xl/sharedStrings.xml`, and relationship files.

### How to Use

1. Copy scripts to workspace: `cp scripts/xlsx_unpack.py .` (repeat for each needed script)
2. Copy template to workspace: `cp -r templates/minimal_xlsx/ .`
3. Load references as needed: `cat references/edit.md`

---

## Dependencies

- **Python**: `openpyxl` (read-only mode), `pandas` (analysis)
- **System**: `unzip`, `zip` (for XML-level editing)
- **Validation**: LibreOffice (`soffice`) for formula recalculation
- **No external libraries needed for XML editing** — standard ZIP + text manipulation
- **Bundled scripts**: Pure Python, no additional pip dependencies

---

## Attribution

This skill incorporates knowledge from:
- [MiniMax Office Skills](https://github.com/MiniMax-AI/skills) (MIT) — XML unpack/edit/repack pattern, formula integrity rules, helper scripts architecture
- [OpenAI Skills](https://github.com/openai/skills) (Apache-2.0) — Financial formatting conventions, IB layout standards, formula hygiene rules, cell color conventions
