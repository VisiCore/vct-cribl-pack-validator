# VCT Cribl Pack Validator

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that validates Cribl `.crbl` pack files against pack standards and produces structured review reports.

## Installation

### From GitHub

```bash
# Add this repo as a plugin marketplace source
claude plugins marketplace add VisiCore/vct-cribl-pack-validator

# Install the plugin
claude plugins install validate-pack
```

### For Local Development

```bash
git clone https://github.com/VisiCore/vct-cribl-pack-validator
cd vct-cribl-pack-validator
claude plugins marketplace add .
claude plugins install validate-pack --scope local
```

### Verify

```bash
claude plugins list
```

## Usage

```
/validate-pack my-pack.crbl
/validate-pack /path/to/your-pack.crbl
```

The skill unpacks the `.crbl`, runs all validation checks, outputs a structured report, saves it to `./reports/`, and cleans up automatically.

Reports produce an **APPROVED**, **NEEDS CHANGES**, or **BLOCKED** verdict.

## What It Validates

- **Naming & ID conventions** -- pack IDs, pipeline names, route names
- **Routing configuration** -- `__group` vs `input_id`, `data_type` fields
- **Source & destination review** -- hardcoded paths, duplicate destinations
- **Pipeline & function audit** -- ordering, redundancy, risky logic
- **PII & security check** -- mask coverage, credential hygiene
- **Test coverage gap analysis** -- what tests exist and what's missing

Standards reference:
- [Cribl Stream Pack Standards](https://docs.cribl.io/stream/packs-standards/)
- [Cribl Edge Pack Standards](https://docs.cribl.io/edge/packs-standards/)

## Plugin Structure

```
vct-cribl-pack-validator/
├── .claude-plugin/
│   └── plugin.json               ← plugin manifest
├── skills/
│   └── validate-pack/
│       └── SKILL.md              ← skill definition (single source of truth)
├── CLAUDE.md                     ← repo maintenance context
└── README.md                     ← this file
```

## Contributing

All validation logic lives in `skills/validate-pack/SKILL.md`. Update checks, report format, or standards there -- no duplication to keep in sync.

Pull requests welcome for additional validation checks and improvements.

## Background

This tool addresses the bottleneck of manual pack review. Cribl's pack ecosystem is growing and manual review doesn't scale. This gives reviewers an automated first pass so human time is focused on edge cases and judgment calls.
