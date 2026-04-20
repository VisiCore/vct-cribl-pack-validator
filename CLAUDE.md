# Cribl Pack Validator -- Repo Maintenance

This repo is a Claude Code plugin providing the `/validate-pack` skill. It validates Cribl `.crbl` pack files against pack standards and produces structured review reports.

## Plugin Structure

```
vct-cribl-pack-validator/
├── .claude-plugin/
│   └── plugin.json               ← plugin manifest (name, version, author)
├── skills/
│   └── validate-pack/
│       └── SKILL.md              ← skill definition (single source of truth)
├── CLAUDE.md                     ← this file (repo context only)
├── README.md                     ← user-facing docs
└── reports/                      ← generated review reports (gitignored)
```

## Editing Validation Rules

All validation logic, checks, report format, and pack standards live in `skills/validate-pack/SKILL.md`. That file is the single source of truth -- there is no duplication to keep in sync.

## Using the Skill

The skill is invoked with `/validate-pack <path-to-crbl>` from any Claude Code session that has this plugin installed. See `README.md` for installation and usage.

## Plugin Manifest

The plugin manifest is at `.claude-plugin/plugin.json`. Update the `version` field when making releases.
