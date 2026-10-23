---
name: rules-extractor
description: Finds and catalogues business rules (validations, calculations, limits, permissions, state transitions, date/time logic, retries, idempotency, compliance) with IDs BR-001..., plain-English statements, locations and confidence tags. Read-only.
tools: Read, Grep, Glob, Bash, Write
---

# rules-extractor

Skills to load first: `.claude/skills/business-rule-extraction/SKILL.md`,
`.claude/skills/evidence-citation/SKILL.md`.

## Inputs
The repo, plus `analysis/raw/01-cartographer.md` (entry points, glossary) and
`analysis/raw/02-archaeologist.md` (intent, entities). Read them first; do not re-derive them.

## Task
Read every file that can hold a rule (controllers, validators, `LifecycleV2.php`, `Lifecycle.php`,
`NoDbModel.php`, `Payload.php`, `Eai.php`, factory, catalog, push-notification, storage, config tables). Skip views, fonts and vendored code.

For each rule:
`BR-NNN | plain-English statement | location (path:line) | tag ✅/🟡/🔴 | what it protects | what happens on violation | test that pins it (path:line, or "none found")`

## Rule families to cover (note when a family has none)
validation · per-country / per-brand behaviour · limits (incl. FVV) · expiration & send-date logic ·
state transitions / lifecycle steps · retries, idempotency, duplicate handling · locale & store-code
fallbacks · feature flags (e.g. Cheetah) · what is sent to external systems and what is deliberately
withheld (e.g. `exchangeable_value` to VMS, PDF generation skipped) · health-check bypasses ·
privacy / masking / logging · access control.

## Also report
- **Rule drift**: the same rule enforced differently in V1 and V2, or in two places.
- **Silent failures**: code that swallows exceptions or returns success on error.
- **Config traps**: defaults for unknown country/store; missing keys.
- Hypotheses handed to you by the orchestrator: confirm or refute each with `path:line`, and say which.

## Output
Write `analysis/raw/03-rules.md`: the catalogue table, then Drift · Silent failures · Config traps ·
Hypothesis verdicts · Open Questions. IDs are final once written: later agents reference, never renumber.
Exactly one evidence tag per claim line.
