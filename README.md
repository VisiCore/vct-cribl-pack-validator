# VCT Cribl Pack Validator

AI-powered Cribl pack review and validation using Claude Code. Point it at a `.crbl` file, get a structured review report back.

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

Clone the repo:

```bash
git clone https://github.com/VisiCore/vct-cribl-pack-validator
cd vct-cribl-pack-validator
```

To use the `/validate-pack` skill from **any directory**, add this repo's skills to your user-level Claude Code settings (`~/.claude/settings.json`):

```json
{
  "skills": [
    "/path/to/vct-cribl-pack-validator/.claude/skills"
  ]
}
```

---

## Usage

From any Claude Code session:

```
/validate-pack my-pack.crbl
/validate-pack /path/to/your-pack.crbl
```

The skill unpacks the `.crbl`, runs all validation checks, outputs the report, saves it to `./reports/`, and cleans up automatically.

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

## Pack Inventory
- Sources: 2 (http-in, file-monitor)
- Pipelines: 3 (parse-http, mask-pii, enrich)
- Routes: 4
- Destinations: 1
- Functions used: Eval, Mask, Regex Extract, Drop

## Passed (8)
...

## Warnings — Non-Blocking (2)
...

## Required Changes — Blocking (1)
- Routing uses input_id: Route 'to-splunk' filters on __inputId.
  Change to __group to prevent breaks on source rename.

## Test Coverage
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
│   ├── settings.local.json           ← project-level permission allowlist
│   └── skills/
│       └── validate-pack/
│           └── SKILL.md              ← skill definition (single source of truth)
├── CLAUDE.md                         ← repo context (no validation logic)
├── README.md                         ← this file
└── reports/                          ← generated review reports
```

---

## Contributing

All validation logic lives in a single file: `.claude/skills/validate-pack/SKILL.md`. Update checks, report format, or standards there — no duplication to keep in sync.

Pull requests welcome for additional validation checks and improvements.

---

## Background

This tool was built to address the bottleneck of manual pack review. Cribl's pack ecosystem is growing significantly and the manual review process (naming script + human eyeballs) doesn't scale. This gives reviewers an automated first pass so human review time is focused on edge cases and judgment calls.

Pairs well with the Cribl platform's March 2026 update introducing global destination access from packs and pack-to-pack routing.
