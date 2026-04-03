---
name: bank-statement-converter
display_name: Bank Statement Converter
version: "1.0.0"
description: >-
  Convert PDF bank and credit card statements into CSV and QBO files.
  Processes each file independently with mandatory validation against
  statement summaries before proceeding. Supports any bank format.
  Keywords: bank statement, PDF, CSV, QBO, QuickBooks, OFX, accounting, finance.
tags:
  - accounting
  - finance
  - pdf
  - csv
  - quickbooks
allowed-tools: >-
  Read Write Edit Bash
---

# Bank Statement Converter

Converts PDF bank and credit card statements into structured CSV and QBO (QuickBooks) files. Each PDF is processed independently with full validation before moving to the next.

---

## Mandatory Rules

### Never Do

1. **Never write a single script to batch-process multiple PDFs** — even if you believe the formats are identical.
2. **Never assume two PDFs share the same format** — even from the same bank.
3. **Never skip the validation step.**
4. **Never proceed to the next file before validation passes.**

### Always Do

Complete the full pipeline for each PDF before starting the next:

```
Step 1: Read PDF and extract text
  ↓
Step 2: Analyze format → parse transactions line by line
  ↓
Step 3: Generate CSV (dates in YYYY-MM-DD)
  ↓
Step 4: ★ Full CSV Validation ★ — must pass before continuing
  ↓
Step 5: Generate QBO file
  ↓
Step 6: ★ Full QBO Validation ★ — must pass before continuing
  ↓
Step 7: Save files to the same directory as the source PDF
  ↓
Only after all validations pass → start next PDF
```

---

## Step 1: Extract Text from PDF

Use `pdfplumber` to read the PDF and extract text:

```python
import pdfplumber

with pdfplumber.open("statement.pdf") as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        print(text)
```

## Step 2: Analyze Format and Parse Transactions

**Every PDF may have a different layout. Always analyze before parsing.**

### 2.1 Examine the PDF Structure

After extracting text, observe:
- How transactions are arranged (table, list, mixed)
- Date position and format
- Amount layout (single column with +/-, two separate columns)
- Whether descriptions wrap across multiple lines
- Whether summary/total rows need to be excluded

### 2.2 Standard Fields

Bank statements typically contain:

| Field | Common Labels |
|-------|---------------|
| Date | Date, Trans Date, Posted |
| Description | Description, Memo, Details, Payee |
| Credit | Credit, Deposits, Money In |
| Debit | Debit, Withdrawals, Money Out |

### 2.3 Determine Amount Direction

- **Two-column format** (separate Credit/Debit columns): map directly.
- **Single-column format** (signed amounts):
  - Positive or unsigned → Credit (income)
  - Negative or parenthesized → Debit (expense)

### 2.4 Parsing Principle

**Parse based on the actual format of the current PDF.** Do not rely on hardcoded patterns or regular expressions from other statements.

## Step 3: Standardize Dates and Generate CSV

### Date Normalization

Convert all dates to `YYYY-MM-DD` (ISO 8601):

| PDF Format | Standardized | Example |
|------------|-------------|---------|
| `1 February` / `Feb 1` | `YYYY-02-01` | `2025-02-01` |
| `Nov 03, 2025` | `2025-11-03` | `2025-11-03` |
| `03/15/2025` (MM/DD/YYYY) | `2025-03-15` | `2025-03-15` |
| `15/03/2025` (DD/MM/YYYY) | `2025-03-15` | `2025-03-15` |
| `03-15-25` | `2025-03-15` | `2025-03-15` |

### When the Year Is Missing

If the PDF only shows month and day (e.g., `Jan 15`):

1. **Check the PDF header** — usually states the period, e.g., "Statement Period: January 1 – January 31, 2025"
2. **Check nearby rows** — adjacent transactions may include full dates
3. **Check first/last entries** — summary sections often include the year
4. **Handle year boundaries** — if December is followed by January, increment the year

**Do not assume all transactions fall in the same year.** Cross-year statements (Dec–Jan) are common.

### CSV Format

```python
import csv

with open("transactions.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["Date", "Description", "Credit", "Debit"])
    for t in transactions:
        writer.writerow([t["date"], t["description"], t["credit"], t["debit"]])
```

**Rules:**
- Columns: `Date`, `Description`, `Credit`, `Debit`
- Dates: `YYYY-MM-DD`
- Amounts: numbers only, no currency symbols
- Empty credit or debit: leave blank
- Descriptions containing commas must be quoted (handled by `csv.writer`)

## Step 4: CSV Validation (Mandatory)

### A1. Transaction Count

- [ ] Count rows in the CSV
- [ ] Cross-reference every transaction against the PDF
- [ ] Confirm the counts match

### A2. Amount Totals

- [ ] Sum all Credit values in the CSV
- [ ] Sum all Debit values in the CSV
- [ ] Compare against the PDF summary totals (usually at the top or bottom of the statement)
- [ ] CSV Credit total ≈ PDF "Total Deposits / Money In"
- [ ] CSV Debit total ≈ PDF "Total Withdrawals / Money Out"

**Tolerance: ≤ $0.01. Larger discrepancies require investigation.**

### A3. Per-Transaction Check

- [ ] All dates are `YYYY-MM-DD`
- [ ] Dates are correct (e.g., February = 02, not 01)
- [ ] Descriptions are complete (no truncation)
- [ ] Credit/Debit amounts are positive numbers in the correct column

### If Validation Fails

1. Stop — do not proceed to QBO generation
2. Re-analyze the PDF format from Step 1
3. Fix the issue and regenerate CSV
4. Re-run full validation

## Steps 5 & 6: Generate and Validate QBO

QBO is QuickBooks' bank statement import format based on OFX/SGML.

```python
import csv
from datetime import datetime

def csv_to_qbo(csv_file, qbo_file):
    transactions = []
    with open(csv_file, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            transactions.append(row)

    # Date range from all transactions (not just first/last row)
    dates = [datetime.strptime(t["Date"], "%Y-%m-%d") for t in transactions]
    dt_start = min(dates).strftime("%Y%m%d")
    dt_end = max(dates).strftime("%Y%m%d")

    qbo_lines = [
        "OFXHEADER:100",
        "DATA:OFXSGML",
        "VERSION:102",
        "<OFX>",
        "<BANKTRANLIST>",
        f"<DTSTART>{dt_start}</DTSTART>",
        f"<DTEND>{dt_end}</DTEND>",
    ]

    for t in transactions:
        credit = float(t["Credit"]) if t.get("Credit", "").strip() else 0
        debit = float(t["Debit"]) if t.get("Debit", "").strip() else 0
        amount = credit - debit
        date_str = t["Date"].replace("-", "")

        desc = t["Description"].upper()
        if any(kw in desc for kw in ["PAYMENT", "WITHDRAWAL", "DEBIT", "PURCHASE", "CHARGE"]):
            trn_type = "DEBIT"
        elif any(kw in desc for kw in ["TRANSFER IN", "CREDIT", "DEPOSIT", "SALARY", "INCOME"]):
            trn_type = "CREDIT"
        else:
            trn_type = "GEN"

        qbo_lines.extend([
            "<STMTTRN>",
            f"<TRNTYPE>{trn_type}</TRNTYPE>",
            f"<DTPOSTED>{date_str}</DTPOSTED>",
            f"<TRNAMT>{amount:.2f}</TRNAMT>",
            f"<NAME>{t['Description']}</NAME>",
            f"<MEMO>{t['Description']}</MEMO>",
            "</STMTTRN>",
        ])

    qbo_lines.extend(["</BANKTRANLIST>", "</OFX>"])

    with open(qbo_file, "w", encoding="utf-8") as f:
        f.write("\n".join(qbo_lines))
```

### QBO Field Reference

| Field | Description | Example |
|-------|-------------|---------|
| OFXHEADER | OFX version header | `OFXHEADER:100` |
| DTSTART/DTEND | Date range (YYYYMMDD) | `20250101` |
| DTPOSTED | Transaction date (YYYYMMDD) | `20250115` |
| TRNAMT | Amount (positive = income, negative = expense) | `-25.00` |
| TRNTYPE | Type: DEBIT, CREDIT, or GEN | `DEBIT` |

### QBO Validation

- [ ] QBO file was generated successfully
- [ ] Transaction count matches the CSV
- [ ] All dates are YYYYMMDD (8 digits)
- [ ] Amounts have correct signs (income positive, expense negative)
- [ ] DTSTART/DTEND range is correct

---

## Output Files

### Location

Save output files in the **same directory** as the source PDF.

### Naming Convention

| Input | CSV Output | QBO Output |
|-------|-----------|------------|
| `acme_bank_jan.pdf` | `acme_bank_jan_transactions.csv` | `acme_bank_jan.qbo` |
| `chase_checking_q1.pdf` | `chase_checking_q1_transactions.csv` | `chase_checking_q1.qbo` |

- **CSV**: append `_transactions` suffix to the original filename
- **QBO**: use the original filename with `.qbo` extension

---

## Troubleshooting

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Transaction count too low | Missed rows or multi-line descriptions not merged | Re-examine PDF text line by line |
| Amount totals mismatch | Wrong amount direction or missing transactions | Compare against PDF summary totals |
| Wrong date format | Non-ISO format or incorrect month mapping | Verify all dates are YYYY-MM-DD |
| Description truncated | Multi-line description only captured first line | Check if descriptions span multiple lines |
| QBO import fails | Date format error or missing tags | Confirm YYYYMMDD format and all required fields |

---

## Dependencies

- `pdfplumber` — PDF text extraction
- Python standard library: `csv`, `datetime`

---

## Multi-File Processing Summary

```
file1.pdf → extract → analyze → parse → CSV → [Validate] → QBO → [Validate] → ✓ saved
                                                                                  ↓
file2.pdf → extract → analyze → parse → CSV → [Validate] → QBO → [Validate] → ✓ saved
                                                                                  ↓
file3.pdf → extract → analyze → parse → CSV → [Validate] → QBO → [Validate] → ✓ saved
```

**Each file must complete all validations (✓) before processing the next.**
