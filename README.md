# VCT Cribl Pack Validator

AI-powered Cribl pack review and validation using Claude Code. Point it at a `.crbl` file, get a structured review report back — no prompting required.

---

## What It Does

Validates Cribl packs against official pack standards including:

- **Naming & ID conventions** — pack IDs, pipeline names, route names
- **Routing configuration** — `__group` vs `input_id`, `data_type` fields
- **Source & destination review** — hardcoded paths, duplicate destinations
- **Pipeline & function audit** — ordering, redundancy, risky logic
- **PII & security check** — mask coverage, credential hygiene
- **Test coverage gap analysis** — what tests exist and what's missing

---

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- A Cribl `.crbl` pack file to validate

---

## Setup

```bash
git clone https://github.com/VisiCore/vct-cribl-pack-validator
cd vct-cribl-pack-validator
```

No additional configuration needed.

---

## Usage

There are two ways to use the validator — both produce the same validation checks and report format.

### Option 1: Skill (recommended)

The `/validate-pack` skill is an on-demand command you invoke explicitly. Start a Claude Code session in this directory and run:

```
/validate-pack my-pack.crbl
/validate-pack /path/to/your-pack.crbl
```

This is the recommended approach — the skill only loads when invoked, keeping your context window free for other work.

### Option 2: CLAUDE.md (auto-loaded)

The `CLAUDE.md` in the repo root is automatically loaded into every Claude Code session in this directory. Just describe what you want naturally:

```
analyze my-pack.crbl
analyze /path/to/your-pack.crbl
```

This works because the validation instructions are always present in context. The tradeoff is that the instructions consume context window space even when you're not validating a pack.

### Which should I use?

| | Skill (`/validate-pack`) | CLAUDE.md (natural language) |
|---|---|---|
| Trigger | Explicit `/validate-pack <file>` | Any message mentioning a `.crbl` file |
| Context cost | Zero until invoked | Always loaded |
| Best for | Focused validation runs | Working in this repo full-time |

Both approaches run identical validation checks and produce the same report format.

---

## Report Output

Reports are saved to `./reports/<pack_id>-<date>.md` and look like this:

```
════════════════════════════════════════════
  CRIBL PACK REVIEW REPORT
  Pack: cc-edge-http-collector-io
  File: example-http-collector.crbl
  Date: 2026-03-05
════════════════════════════════════════════

## 📦 Pack Inventory
- Sources: 2 (http-in, file-monitor)
- Pipelines: 3 (parse-http, mask-pii, enrich)
- Routes: 4
- Destinations: 1
- Functions used: Eval, Mask, Regex Extract, Drop

## ✅ Passed (8)
...

## ⚠️ Warnings — Non-Blocking (2)
...

## ❌ Required Changes — Blocking (1)
- Routing uses input_id: Route 'to-splunk' filters on __inputId. 
  Change to __group to prevent breaks on source rename.

## 🧪 Test Coverage
...

════════════════════════════════════════════
  VERDICT: NEEDS CHANGES
════════════════════════════════════════════
```

---

## Pack Standards Enforced

| Standard | Rule |
|---|---|
| Pack ID format | `cc-edge-<source>-io` for edge packs, `cc-stream-<source>-io` for stream packs |
| Pipeline naming | Never `main` — always descriptive |
| Routing field | Always `__group`, never `input_id` |
| Source metadata | `data_type` field required on all sources |
| Destinations | Prefer global destinations (post March 2026) |
| File paths | Always environment variables, never hardcoded |
| PII | Must be masked before any destination |

---

## Repo Structure

```
vct-cribl-pack-validator/
├── .claude/
│   └── skills/
│       └── validate-pack.md   ← /validate-pack skill definition
├── CLAUDE.md                  ← auto-loaded validation instructions
├── README.md                  ← this file
├── reports/                   ← generated review reports saved here
└── tmp/                       ← temporary extraction dir (auto-created, auto-cleaned)
```

Place `.crbl` files anywhere — just provide the path when you run an analysis.

---

## Background

This tool was built to address the bottleneck of manual pack review. Cribl's pack ecosystem is growing significantly and the manual review process (naming script + human eyeballs) doesn't scale. This gives reviewers an automated first pass so human review time is focused on edge cases and judgment calls.

Pairs well with the Cribl platform's March 2026 update introducing global destination access from packs and pack-to-pack routing.

---

## Contributing

Validation rules and the report format are defined in two places that should be kept in sync:

- `CLAUDE.md` — auto-loaded project instructions
- `.claude/skills/validate-pack.md` — on-demand skill

When updating validation checks or the report format, update both files.

Pull requests welcome for additional validation checks and improvements.
