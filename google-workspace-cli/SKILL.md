---
name: google-workspace-cli
display_name: Google Workspace CLI
version: "1.0.0"
description: >-
  Use Google Workspace services (Calendar, Gmail, Drive, Sheets, Docs, Tasks)
  via the gws CLI. Covers installation, authentication for personal and
  Workspace accounts, helper commands, schema discovery, and safety practices.
  Keywords: Google, Calendar, Gmail, Drive, Sheets, Docs, gws, Workspace, CLI.
tags:
  - google
  - calendar
  - gmail
  - drive
  - productivity
  - cli
allowed-tools: >-
  Read Bash
---

# Google Workspace CLI (`gws`)

A single CLI for all Google Workspace APIs — Calendar, Gmail, Drive, Sheets, Docs, Chat, Tasks, and more. Output is structured JSON by default, designed for AI agent consumption.

**Repository**: https://github.com/googleworkspace/cli
**License**: Apache-2.0
**Works with**: Google Workspace accounts AND personal @gmail.com accounts

---

## Installation

```bash
# npm (recommended — bundles pre-built native binaries)
npm install -g @googleworkspace/cli

# Homebrew (macOS/Linux)
brew install googleworkspace-cli
```

Verify: `gws --version`

---

## Authentication

### Quick Setup (if `gcloud` is installed)

```bash
gws auth setup     # creates Cloud project, enables APIs, logs in
```

### Manual Setup (personal @gmail.com or custom project)

1. Open [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project
3. Go to **APIs & Services > OAuth consent screen**
   - Choose **External**, set to **Testing** mode
   - **Add yourself as a Test User** (critical — without this, login fails)
4. Go to **Credentials > Create Credentials > OAuth client ID**
   - Application type: **Desktop app**
   - Download the JSON file
5. Save it as `~/.config/gws/client_secret.json`
6. Run:
   ```bash
   gws auth login -s calendar,gmail,drive,sheets,docs
   ```

### Important: Scope Selection for Personal Accounts

Unverified (testing mode) apps are limited to ~25 OAuth scopes. The default `recommended` preset includes 85+ scopes and **will fail**. Always specify only the services you need:

```bash
gws auth login -s calendar,gmail,drive,sheets,docs,tasks
```

### Headless / CI Environments

```bash
# On a machine with a browser:
gws auth export --unmasked > credentials.json

# On the headless machine:
export GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE=/path/to/credentials.json
```

---

## Core Syntax

```bash
gws <service> <resource> <method> [flags]
gws <service> +<helper> [flags]
```

Key flags:
- `--params '{"key": "val"}'` — URL/query parameters
- `--json '{"key": "val"}'` — Request body
- `--fields "field1,field2"` — Field mask (always use to keep output small)
- `--format table|yaml|csv` — Override default JSON output
- `--dry-run` — Validate without calling API
- `--page-all` — Auto-paginate (NDJSON output)

---

## Calendar

```bash
# View upcoming events
gws calendar +agenda
gws calendar +agenda --today
gws calendar +agenda --week
gws calendar +agenda --days 3 --format table

# Create an event
gws calendar +insert \
  --summary "Team standup" \
  --start "2026-04-01T09:00:00" \
  --end "2026-04-01T09:30:00"

# Create event with attendees and Google Meet
gws calendar +insert \
  --summary "Design review" \
  --start "2026-04-01T14:00:00" \
  --end "2026-04-01T15:00:00" \
  --attendee alice@example.com \
  --attendee bob@example.com \
  --meet

# Raw API (for advanced queries)
gws calendar events list \
  --params '{"calendarId": "primary", "timeMin": "2026-04-01T00:00:00Z", "maxResults": 10}' \
  --fields "items(id,summary,start,end)"
```

## Gmail

```bash
# Inbox triage
gws gmail +triage
gws gmail +triage --max 5 --query "is:unread from:boss"

# Read a message
gws gmail +read --id MESSAGE_ID

# Send email
gws gmail +send --to alice@example.com --subject "Report" --body "Please review."
gws gmail +send --to alice@example.com --subject "Q1" --body "Attached." -a report.pdf

# Save as draft instead of sending
gws gmail +send --to alice@example.com --subject "Draft" --body "..." --draft

# Reply / Forward
gws gmail +reply --message-id MSG_ID --body "Thanks!"
gws gmail +forward --message-id MSG_ID --to colleague@example.com
```

## Drive

```bash
# List files
gws drive files list \
  --params '{"pageSize": 10}' \
  --fields "files(id,name,mimeType)"

# Search
gws drive files list \
  --params '{"q": "name contains \"Report\"", "pageSize": 10}' \
  --fields "files(id,name,mimeType)"

# Upload
gws drive +upload ./report.pdf
gws drive +upload ./report.pdf --parent FOLDER_ID --name "Q1 Report"
```

## Sheets

```bash
# Read values
gws sheets +read --spreadsheet SPREADSHEET_ID --range "Sheet1!A1:D10"

# Append a row
gws sheets +append --spreadsheet SPREADSHEET_ID --values "Alice,100,true"

# Append multiple rows (JSON)
gws sheets +append --spreadsheet SPREADSHEET_ID \
  --json-values '[["Name","Score"],["Alice","95"],["Bob","87"]]'

# Create new spreadsheet
gws sheets spreadsheets create --json '{"properties": {"title": "Q1 Budget"}}'
```

## Docs

```bash
# Append text to a document
gws docs +write --document DOCUMENT_ID --text "New section content"
```

## Workflow Helpers (Cross-Service)

```bash
gws workflow +standup-report     # Today's meetings + open tasks
gws workflow +meeting-prep       # Next meeting: agenda, attendees, linked docs
gws workflow +weekly-digest      # Weekly summary: meetings + unread count
gws workflow +email-to-task      # Convert Gmail message to Google Tasks entry
```

---

## Schema Discovery

Before calling an unfamiliar API, inspect its schema:

```bash
gws schema calendar.events.insert
gws schema drive.files.list
gws schema sheets.spreadsheets.values.append
```

This shows required parameters, request body fields, and response structure.

---

## Safety Practices

1. **Always use `--fields`** when listing or getting resources — Workspace APIs return massive JSON blobs that waste context window.
2. **Use `--dry-run`** before mutating operations (create, update, delete) to validate the payload.
3. **Confirm with the user** before sending emails, creating events with attendees, or deleting files.
4. **Never output tokens or credentials** in responses.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | API error (4xx/5xx) |
| 2 | Auth error — re-run `gws auth login` |
| 3 | Validation error — check params |

## Shell Tips

- Wrap `--params` and `--json` values in **single quotes** (they contain double quotes)
- Sheet ranges with `!` in zsh: use double quotes — `--range "Sheet1!A1:D10"`

---

## Dependencies

- `@googleworkspace/cli` (npm) or `googleworkspace-cli` (Homebrew)
- A Google account (personal @gmail.com or Workspace)
- A Google Cloud project with OAuth credentials
