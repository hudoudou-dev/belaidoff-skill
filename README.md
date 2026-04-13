<div align="center">

# belaidoff.skill

> *"When layoff comes, you are not alone in the fight. Let AI be your most powerful rights protection assistant."*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

Are you facing layoff and don't know how to assert your rights?<br>
Are you worried about being exploited by the company and don't know how to negotiate?<br>
Do you need psychological support to get through this difficult period?<br>
Do you need legal knowledge to protect your legitimate rights and interests?<br>
Do you need to reduce information asymmetry to communicate with the company?<br>

**Let belaidoff.skill be your rights protection assistant, providing professional legal advice and negotiation strategies!**

**May every laid-off worker be treated with kindness. Layoff is not terrible, it's just the beginning of the next chapter of life.**

It is a mirror of **[layoff.skill](https://github.com/hudoudou-dev/layoff-skill)** (by hudoudou-dev).

<br>

[Supported Sources](#supported-data-sources) · [Install](#install) · [Usage](#usage) · [Demo](#demo) · [Detailed Install](INSTALL.md)

[**中文**](README_ZH.md)

</div>

---

Upload your personal information and relevant layoff materials (work emails, Feishu/DingTalk chat records, contracts, etc.) to generate a **personalized layoff response AI Skill** that helps you calculate compensation, develop negotiation strategies, provide psychological support, and prepare legal documents.

Created by [@hudoudou-dev](https://github.com/hudoudou-dev) | Powered by hudoudou-personal

## Supported Data Sources

> **Note**: This project is modified based on colleague.skill, specifically designed to provide rights protection and psychological support for employees facing layoff.

> This is currently an alpha version of belaidoff.skill — more sources will be supported in the future, stay tuned!

| Source | Messages | Docs / Wiki | Spreadsheets | Notes |
|--------|:--------:|:-----------:|:------------:|-------|
| Feishu (auto) | ✅ API | ✅ | ✅ | Just enter a name, fully automatic |
| DingTalk (auto) | ⚠️ Browser | ✅ | ✅ | DingTalk API doesn't support message history |
| PDF | — | ✅ | — | Manual upload |
| Images / Screenshots | ✅ | — | — | Manual upload |
| Feishu JSON export | ✅ | ✅ | — | Manual upload |
| Email `.eml` / `.mbox` | ✅ | — | — | Manual upload |
| Markdown | ✅ | ✅ | — | Manual upload |
| Paste text directly | ✅ | — | — | Manual input |

---

## Install

### Claude Code

> **Important**: Claude Code looks for skills in `.claude/skills/` at the **git repo root**. Make sure you run this in the right place.

```bash
# Install to current project (run at git repo root)
mkdir -p .claude/skills
git clone https://github.com/hudoudou-dev/belaidoff-skill .claude/skills/create-belaidoff

# Or install globally (available in all projects)
git clone https://github.com/hudoudou-dev/belaidoff-skill ~/.claude/skills/create-belaidoff
```

### OpenClaw

```bash
git clone https://github.com/hudoudou-dev/belaidoff-skill ~/.openclaw/workspace/skills/create-belaidoff   
```

### Dependencies (optional)

```bash
pip3 install -r requirements.txt
```

> Feishu/DingTalk auto-collection requires App credentials. See [INSTALL.md](INSTALL.md) for details.

---

## Usage

In Claude Code, type:

```
/create-belaidoff
```

Follow the prompts: enter personal information (name, basic work information, personal status, layoff situation, personal status tags), then choose a data source. All fields can be skipped — even a description alone can generate a Skill.

Once created, invoke the layoff response Skill with `/{slug}`.

### Commands

| Command | Description |
|---------|-------------|
| `/list-belaidoff` | List all layoff response Skills |
| `/{slug}` | Invoke full Skill (Rights Protection Capabilities + Personal Profile) |
| `/{slug}-capabilities` | Rights protection capabilities only |
| `/{slug}-persona` | Personal profile only |
| `/belaidoff-rollback {slug} {version}` | Rollback to a previous version |
| `/delete-belaidoff {slug}` | Delete |

---

## Demo

> Input: `Zhang San, Alibaba 2020-01-01 13 600k/year 325, 32 master married 1 child has mortgage (8000/month), negotiating 2N compensation, has children has mortgage anxious assertive medium risk`

**Scenario 1: Compensation Calculation**

```
User            ❯ How much compensation can I get?
belaidoff.skill ❯ Based on your situation, the calculation is as follows:
                  - Working years: 4 years
                  - Average monthly salary: 50,000 yuan
                  - Economic compensation: 4 × 50,000 = 200,000 yuan
                  - If the company illegally terminates the labor contract, compensation: 2 × 200,000 = 400,000 yuan
                  - Without 30-day advance notice, substitute payment: 50,000 yuan
                  Total maximum: about 450,000 yuan
```

**Scenario 2: Negotiation Strategy**

```
User            ❯ HR said they can only give N+1, how should I negotiate?
belaidoff.skill ❯ I suggest you:
                  1. Collect evidence: labor contract, salary slips, social security records, chat records, etc.
                  2. Emphasize your contributions: list the achievements and results you made for the company
                  3. Cite laws and regulations: According to Article 87 of the Labor Contract Law, illegal termination requires 2N compensation
                  4. Stay calm, communicate rationally, avoid emotional reactions
                  5. Set a bottom line and be prepared for labor arbitration
```

**Scenario 3: Psychological Support**

```
User            ❯ I'm very anxious, I don't know what to do
belaidoff.skill ❯ I understand your feelings, I suggest you:
                  1. Practice deep breathing to relieve anxiety
                  2. Communicate with family and friends to seek emotional support
                  3. Develop a job search plan to shift your focus
                  4. Review past achievements to affirm your value
                  5. Maintain a positive attitude and believe that the future will be better
```

---

## Features

### Generated Skill Structure

Each layoff response Skill has two parts that work together:

| Part | Content |
|------|---------|
| **Part A — Rights Protection Capabilities** | Compensation calculation, negotiation strategies, legal knowledge base, document templates, psychological support |
| **Part B — Personal Profile** | 5-layer structure: Core demands → Mental state → Family burden → Communication style → Risk tolerance |

Execution: `Receive task → Personal Profile judges needs and mental state → Rights Protection Capabilities provide specific solutions → Output combining user's communication style and risk tolerance`

### Supported Tags

- **Family burden**: Has children · Has mortgage · Financial pressure · Family breadwinner · No burden
- **Mental state**: Anxious · Angry · Depressed · Fearful · Calm and rational · Optimistic and positive
- **Communication style**: Assertive · Gentle negotiation · Reserved · Aggressive · Evidence-based
- **Risk tolerance**: High risk · Medium risk · Low risk · Conservative · Aggressive

### Evolution

- **Append files**: Auto-analyze delta, merge into relevant sections, never overwrite existing conclusions
- **Conversation correction**: Say "it should be xxx", write to Correction layer, take effect immediately
- **Version control**: Auto-archive on every update, support rollback to any previous version

---

## Project Structure

This project follows the [AgentSkills](https://agentskills.io) open standard. The entire repo is a skill directory:

```
create-belaidoff/
├── SKILL.md              # Skill entry point (official frontmatter)
├── prompts/              # Prompt templates
│   ├── intake.md         #   Dialogue-based info collection
│   ├── capabilities_analyzer.md # Rights protection capabilities extraction
│   ├── capabilities_builder.md # Rights protection capabilities generation template
│   ├── persona_analyzer.md # Personal profile extraction
│   ├── persona_builder.md # Personal profile 5-layer structure template
│   ├── merger.md         # Incremental merge logic
│   └── correction_handler.md # Conversation correction handler
├── tools/                # Python tools
│   ├── feishu_auto_collector.py  # Feishu auto-collector
│   ├── feishu_browser.py         # Feishu browser method
│   ├── feishu_mcp_client.py      # Feishu MCP method
│   ├── dingtalk_auto_collector.py # DingTalk auto-collector
│   ├── email_parser.py           # Email parser
│   ├── skill_writer.py           # Skill file management
│   └── version_manager.py        # Version archive & rollback
├── belaidoff/           # Generated layoff response Skills (gitignored)
├── docs/PRD.md
├── requirements.txt
└── LICENSE
```

---

## Notes

- **Source material quality = Skill quality**: chat logs + contract documents > manual description only
- Prioritize collecting: chat records with HR/management, labor contracts, salary slips, social security records, text versions of verbal compensation communications, etc.
- Feishu auto-collection requires adding the App bot to relevant group chats
- This is currently a demo version — please file issues if you find bugs!
- The information provided by this project is for reference only, please consult a professional lawyer for specific legal issues

---

## Disclaimer

This project aims to provide information and suggestions, not legal advice. All content is for reference only, please consult a professional lawyer for specific legal issues. The project does not assume responsibility for any consequences arising from the use of this project.

---

## Acknowledgements

This project's architecture was inspired by:

- **[colleague.skill](https://github.com/titanwings/colleague-skill)** (by titanwings) — Pioneered the dual-layer architecture of "distilling a person into an AI Skill"

---

## Star History

<a href="https://www.star-history.com/?repos=hudoudou-dev%2Fbelaidoff-skill&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&legend=top-left" />
 </picture>
</a>

---

<div align="center">

MIT License © [hudoudou-dev](https://github.com/hudoudou-dev)

</div>
