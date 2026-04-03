# Forgeuno

Open-source data and skills for AI agents.

**Website**: [forgeuno.com](https://forgeuno.com) — browse models, tools, and skills.

## Skills

Each skill is a self-contained directory following the [Agent Skills specification](https://agentskills.io/specification).

| Skill | Category | Description |
|-------|----------|-------------|
| [bank-statement-converter](skills/bank-statement-converter/) | accounting | Convert PDF bank statements to CSV and QBO |
| [google-workspace-cli](skills/google-workspace-cli/) | productivity | Google Workspace via gws CLI (Calendar, Gmail, Drive) |
| [industry-briefing](skills/industry-briefing/) | content | Research and write industry briefings |
| [office-docx](skills/office-docx/) | documents | Word documents with .NET OpenXML SDK |
| [office-xlsx](skills/office-xlsx/) | documents | Excel with XML-level editing for formula integrity |
| [office-pdf](skills/office-pdf/) | documents | PDF generation, reading, and Markdown→PDF |
| [office-pptx](skills/office-pptx/) | documents | PowerPoint with PptxGenJS and design system |

### Installation

```bash
# Via skills CLI
npx skills add clawuno/forgeuno --path skills/office-xlsx

# Manual
git clone https://github.com/clawuno/forgeuno.git
cp -r forgeuno/skills/office-xlsx ~/.agents/skills/
```

## License

MIT
