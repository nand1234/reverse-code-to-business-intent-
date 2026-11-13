---
name: report-writing
description: The output template and length limits for the final business-intent report. Use when assembling analysis/business-intent-report.md.
---

# Report writing

Write for a Product Owner and a QA lead: business language first, technical detail second.
Main report ≈ **2–3 pages**. Push detail to `analysis/raw/` and link to it. Every claim line keeps
its evidence tag and `path:line` (see `evidence-citation`); one tag per claim line.

## Template (in this order)

### 1. Executive Summary (≤ 6 lines)
What the product is, who it serves, the problem it solves, the single biggest customer risk.

### 2. Business Intent & Value
Problem · Personas · Value proposition · Business-model hints (subscriptions, transactions, …),
each with a confidence tag.

### 3. Real-World Business Flows
For each key journey (3–7): purpose, a Mermaid diagram, the happy path, the top unhappy paths,
systems touched. Keep diagrams ≤ 12 nodes; full versions go in `raw/04-flows.md`.

### 4. Business Rules Catalogue
Table: `BR-ID | Rule (plain English) | Location | Confidence | Customer impact if broken`.
Show the highest-risk rules; link to `raw/03-rules.md` for the full list.

### 5. Customer Impact Map
Table: `Flow/Rule | Failure scenario | Who is hurt and how | Severity | Detectability`.

### 6. Key Testing Focus (risk-based)
- Top 10 risks ranked: `Risk score | BR-ID | Pyramid layer | Given/When/Then`
- Explicitly NOT testing (and why)
- Coverage gaps found in existing tests

### 7. Open Questions (for Product/Business)
Every 🔴 item phrased as a question, with an owner (PO / Dev / QA / Ops) and why the answer
matters to customers.

### 8. Recommendations
Top 5 improvements (product, flow, resilience, observability, testability), each rated by
impact and effort (H/M/L).

### 9. Overall Customer Impact Verdict (≤ 5 lines)
How healthy the customer experience is today, the biggest threat, and the one thing to fix first.

### Appendix
System map, glossary, the critic's notes (including any objections left open after the revision round).

## Closing
End with a **confidence summary**: % of ✅ / 🟡 / 🔴 claim lines. Compute it by counting the tag
emoji in the final file (`grep -o`), never estimate. State what was skipped and that the existing
test suite was read but not executed. Credential-shaped literals found in committed config are reported
by file:line only — never reproduce the values.

## Style
- Tables and bullets over prose. No marketing language. No speculation outside Open Questions.
- Name the customer harm in plain words ("the customer never receives their order"), not the exception class.
