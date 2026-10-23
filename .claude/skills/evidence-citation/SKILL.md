---
name: evidence-citation
description: Cite every claim as path:line (or commit SHA) and tag it CONFIRMED / INFERRED / SPECULATIVE. Use whenever you write a finding about what this code does or why.
---

# Evidence & citation

Every claim line carries **a citation and exactly one tag**. Counting tags later only means
something if each claim line has one tag, so never stack two, never leave a claim untagged.

## Tags
- ✅ **CONFIRMED** — explicit in code, tests, schema, config or docs. You read the line.
- 🟡 **INFERRED** — implied by naming, structure or data flow. State what it was inferred from.
- 🔴 **SPECULATIVE** — a guess. It MUST also appear in your "Open Questions" section.

## Citation format
- Code: `app/models/LifecycleV2.php:120` or a range `:120-148`.
- History: the short commit SHA, e.g. `(5280193)`.
- Generated or absent evidence: say "no evidence found in `<paths searched>`" — absence is a finding.

## Line shape
`<tag> <claim in business language> — <technical detail> (`path:line`)`

Example: `✅ A voucher number must be 9–12 digits — validator rule (`app/controllers/EvoucherController.php:232`)`

## Rules
- Business language first, technical detail second.
- Never invent business context (customers, volumes, contracts, SLAs). If the code does not say it, it is 🔴.
- Verify a line number by reading the file before citing it; do not cite from memory.
- Prefer one precise citation over three vague ones.
- If two sources contradict each other, report both and tag the claim 🟡 at best.
