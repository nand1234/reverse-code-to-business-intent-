---
name: anti-over-testing
description: What NOT to test, and how to pick the lowest pyramid layer that can catch a risk. Use when proposing tests, to prevent redundant or low-value coverage.
---

# Anti over-testing

## Skip
- Framework / library behaviour (the framework's own validation engine, HTTP client, PDF/barcode/cloud-SDK internals).
- Trivial getters/setters, DTOs and serializable types with no logic.
- Generated or vendored code, fonts, lockfiles, IDE metadata files.
- **Duplicate coverage across pyramid layers.** If a unit test pins the rule, do not re-assert it
  through HTTP, and do not add an E2E for what an integration test already proves.
- UI/document/email **cosmetics** with no business impact (pixel positions, colour hex values),
  *except* where a wrong value is a legal or brand obligation — then check a single marker, not the layout.
- Per-row repetition of config tables: test the lookup/fallback behaviour once, not every country.

## Prefer the lowest layer that can catch the risk
1. **Unit** — a pure rule or calculation (validators, limit math, date logic, locale fallback).
2. **Integration** — a rule that needs collaborators: queue job + storage, HTTP client + fake server, config wiring.
3. **Contract** — shape of payloads exchanged with external systems (the integrations listed in `analysis/raw/01-cartographer.md`).
4. **E2E / smoke** — only for a whole journey where wiring itself is the risk; keep to a handful.

## For each proposed test, answer
- Which BR-ID does it protect, and what is the customer harm if it fails?
- Why is this the lowest layer that can catch it?
- Is it already covered? (cite the existing test file:line)

## Existing-suite review
Report: tests that assert nothing business-relevant, tests duplicating one another, and rules with
**no** test. Find the suite's location and runner in `analysis/raw/01-cartographer.md`; do not try to run it.
Judge coverage by reading test names and bodies against the code.
