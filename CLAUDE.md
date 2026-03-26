# Cribl Pack Validator — Repo Maintenance

This repo contains the `/validate-pack` skill for Claude Code. The skill validates Cribl `.crbl` pack files against pack standards and produces structured review reports.

## Repo Structure

```
vct-cribl-pack-validator/
├── .claude/
│   ├── settings.local.json        ← project-level permission allowlist
│   └── skills/
│       └── validate-pack/
│           └── SKILL.md           ← skill definition (single source of truth)
├── CLAUDE.md                      ← this file (repo context only)
├── README.md                      ← user-facing docs
└── reports/                       ← generated review reports
```

## Editing Validation Rules

All validation logic, checks, report format, and pack standards live in `.claude/skills/validate-pack/SKILL.md`. That file is the single source of truth — there is no duplication to keep in sync.

## Using the Skill

The skill is invoked with `/validate-pack <path-to-crbl>` from any Claude Code session that has this repo's skills available. See `README.md` for setup and usage details.
