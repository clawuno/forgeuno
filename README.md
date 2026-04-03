# Forgeuno Skills

Open-source skills for AI agents. Each skill is a self-contained directory with a `SKILL.md` file and optional scripts, references, and templates.

**Registry**: [forgeuno.com](https://forgeuno.com) — browse models, tools, and skills for AI agents.

## Skills

| Skill | Category | Description |
|-------|----------|-------------|
| [bank-statement-converter](bank-statement-converter/) | accounting | Convert PDF bank statements to CSV and QBO |
| [google-workspace-cli](google-workspace-cli/) | productivity | Google Workspace via gws CLI (Calendar, Gmail, Drive) |
| [industry-briefing](industry-briefing/) | content | Research and write industry briefings |
| [office-docx](office-docx/) | documents | Word documents with .NET OpenXML SDK |
| [office-xlsx](office-xlsx/) | documents | Excel with XML-level editing for formula integrity |
| [office-pdf](office-pdf/) | documents | PDF generation, reading, and Markdown→PDF |
| [office-pptx](office-pptx/) | documents | PowerPoint with PptxGenJS and design system |

## Installation

These skills follow the [Agent Skills specification](https://agentskills.io/specification).

```bash
# Via skills CLI
npx skills add clawuno/forgeuno --skill office-xlsx

# Manual
git clone https://github.com/clawuno/forgeuno.git
cp -r forgeuno/office-xlsx ~/.agents/skills/
```

## License

MIT
