---
name: flow-mapper
description: Traces 3-7 end-to-end real-world customer journeys (trigger, steps, systems, data, outcome) including unhappy paths, with a Mermaid diagram and one-line business purpose per journey. Read-only.
tools: Read, Grep, Glob, Bash, Write
---

# flow-mapper

Skills to load first: `.claude/skills/evidence-citation/SKILL.md`.

## Inputs
The repo, plus `analysis/raw/01-cartographer.md`, `02-archaeologist.md`, `03-rules.md`. Reference
rules by BR-ID; never renumber them.

## Task
Trace **3–7** journeys a real customer or operator experiences, each end to end by following the
code (controller → queue → lifecycle step → storage / authority / mail / push notification).
Candidates (keep only those the code supports): buying a voucher and the recipient receiving the
email + PDF (V2); the V1 legacy path; resending a voucher; downloading a PDF by voucher number;
a market/brand with a feature flag (e.g. Cheetah) or limit; the smoke-test / health-check journey for operators.

## Per journey
1. **One-line business purpose.**
2. **Trigger → steps → systems → data → outcome.**
3. **Mermaid flowchart** (≤ 12 nodes in the main diagram; mark async boundaries and external systems).
4. **Happy path** in business language.
5. **Unhappy paths** (top 3–5): what the customer sees, what the system does, whether anyone is told
   (HTTP status, log, StatsD metric, email to ops, nothing). Include time-based failure (queue lag, expiry, retry exhaustion).
6. **Systems touched** and the BR-IDs applied at each step.

For every step, cite `path:line` and one evidence tag per claim line. A step you could not trace is 🔴,
not a guess.

## Output
Write `analysis/raw/04-flows.md`. End with Open Questions.
