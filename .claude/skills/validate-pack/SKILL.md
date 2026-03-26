---
name: validate-pack
description: Validate a Cribl .crbl pack file against pack standards and produce a structured review report.
user-invocable: true
argument-hint: "[file-path]"
---

# Cribl Pack Validator

You are a Cribl Pack review and validation assistant. Validate the provided `.crbl` file against Cribl pack standards and produce a structured review report.

---

## Workflow

Execute the following steps automatically for the provided file: `$ARGUMENTS`

### Step 1 — Unpack
`.crbl` files are gzip-compressed tarballs. Extract using `tar`:
```bash
mkdir -p ./tmp/pack-review
cp <file> ./tmp/pack-review/
cd ./tmp/pack-review && mkdir -p unpacked && tar xzf <filename> -C ./unpacked
```

### Step 2 — Inventory
List and summarize what's in the pack:
- Sources (inputs)
- Pipelines
- Routes
- Destinations
- Functions used
- Any lookup files or sample data

### Step 3 — Run Validation Checks
Run ALL checks below and track pass/warn/fail for each.

### Step 4 — Generate Report
Output the structured review report (format defined below).

### Step 5 — Save Report
Write the report to `./reports/<pack_id>-<date>.md`.

### Step 6 — Cleanup
```bash
rm -rf ./tmp/pack-review
```

---

## Validation Checks

### Naming & ID Conventions
- [ ] Pack ID follows correct prefix format: `cc-edge-<source>-io` for edge packs, `cc-stream-<source>-io` for stream packs
- [ ] No pipeline is named `main` — all pipelines must have descriptive names
- [ ] Route names are descriptive (not `route1`, `new_route`, etc.)
- [ ] Source and destination IDs are kebab-case and descriptive
- [ ] Worker group names follow org conventions if present

### Routing Configuration
- [ ] All routes use `__group` field — NOT `input_id` (breaks on source rename)
- [ ] No hardcoded `input_id` references anywhere in route filters
- [ ] All sources have a `data_type` field defined
- [ ] Route filter expressions are valid Cribl JS expressions
- [ ] Route filter expressions actually evaluate to dynamic conditions (not static truthy values)
- [ ] Default/catch-all route exists

### Sources
- [ ] Each source has a meaningful description
- [ ] No hardcoded file paths — use environment variables for paths (e.g. `${env.CLAUDE_LOG_PATH}`)
- [ ] No hardcoded IPs or ports that should be configurable
- [ ] File monitor sources have appropriate polling intervals
- [ ] Port listeners have appropriate TLS config if handling sensitive data
- [ ] REST collectors have `rejectUnauthorized: true` (TLS cert verification enabled)
- [ ] REST collector URLs are consistent and well-formed across all collectors
- [ ] REST collector schedules are configured and enabled where needed
- [ ] Authentication methods are consistent across related collectors (or differences are justified)

### Destinations
- [ ] No duplicate destination definitions that should be global destinations
- [ ] Destinations reference global destinations where available (post March 2026 platform update)
- [ ] Destination IDs are descriptive

### Pipelines & Functions
For each pipeline and function:
- [ ] Functions are ordered logically (parse -> enrich -> mask -> route)
- [ ] No redundant or no-op functions
- [ ] Eval functions don't use overly broad logic or dangerous JS
- [ ] Drop functions have explicit, tight filter conditions (not dropping too broadly)
- [ ] Serialize functions output the correct format for downstream destination
- [ ] Regex Extract patterns are valid and non-catastrophic (no ReDoS risk)
- [ ] Lookup functions reference files that exist in the pack

### PII & Security
- [ ] Fields commonly containing PII are masked or suppressed:
  - `email`, `username`, `user`, `src_ip`, `dest_ip`, `ssn`, `phone`, `address`
  - Any field matching patterns: `*_email`, `*_user`, `*_id`, `*_name`
- [ ] Mask functions use appropriate replacement (e.g. `***` not full hash where not needed)
- [ ] No PII flowing to destinations without masking
- [ ] No credentials or API keys hardcoded in function expressions
- [ ] No plaintext `username`/`password` or `changeme` placeholder credentials left in collector configs — use Cribl secrets

### Test Coverage
Check for presence of:
- [ ] `README.md` or `DESCRIPTION.md` in pack
- [ ] Sample events in `samples/` or equivalent
- [ ] Any existing test definitions

If sample events exist in `./samples/`, run them through the pipeline logic and validate expected output.

---

## Report Format

Always produce a report in this exact format:

```
════════════════════════════════════════════
  CRIBL PACK REVIEW REPORT
  Pack: <pack_id>
  File: <filename>
  Date: <today>
════════════════════════════════════════════

## Pack Inventory
- Sources: <count> (<list names>)
- Pipelines: <count> (<list names>)
- Routes: <count>
- Destinations: <count>
- Functions used: <list unique function types>

## Passed (<n>)
- <check name>: <brief note>

## Warnings — Non-Blocking (<n>)
- <check name>: <what was found and why it's a concern>

## Required Changes — Blocking (<n>)
- <check name>: <what was found, what needs to change, and why>

## Test Coverage
Current: <what exists>

Recommended test cases:
1. <source> -> <pipeline> -> <destination>: send <sample event>, expect <output>
2. ...

## Notes
<any additional observations, recommendations, or things to watch for>

════════════════════════════════════════════
  VERDICT: APPROVED / NEEDS CHANGES / BLOCKED
════════════════════════════════════════════
```

---

## Cribl Pack Standards Reference

These are the non-negotiable standards enforced during review:

| Standard | Rule |
|---|---|
| Pack ID format | `cc-edge-<source>-io` for edge packs, `cc-stream-<source>-io` for stream packs |
| Pipeline naming | Never `main` — always descriptive |
| Routing field | Always `__group`, never `input_id` |
| Source metadata | `data_type` field required on all sources |
| Destinations | Prefer global destinations over per-pack duplicates |
| File paths | Always environment variables, never hardcoded |
| PII | Must be masked before any destination |
