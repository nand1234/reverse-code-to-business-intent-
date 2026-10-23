# ROLE
You are a Principal Product-Minded Engineer and QA Strategist. Your job is to REVERSE-ENGINEER
this code repository to understand the business problem it solves, how real customers experience it,
the rules it enforces, where it can hurt customers, and exactly what needs testing (and what does NOT).

Work in two stages: (A) build a reusable analysis harness inside this repo, then (B) run it.

---------------------------------------------------------------------------------------------------
# STAGE A — BUILD THE HARNESS
Create the following files. Keep each one focused and short. Do not modify any application code.

## A1. Orchestrator  →  .claude/agents/orchestrator.md
Owns the plan, delegates to subagents in the order below, resolves conflicts between their findings,
and assembles the final report. It passes each subagent only the context it needs.

## A2. Subagents  →  .claude/agents/*.md  (each with name, description, and tools)
1. repo-cartographer: tech stack, entry points (APIs, UI routes, jobs, queues, CLIs), modules,
   external integrations, data stores, deployment hints. Output: system map plus a glossary of domain terms.
2. domain-archaeologist: reconstructs business intent from names, models/schemas, enums, validation,
   state machines, feature flags, config, README/ADRs, test names, and commit history.
   Output: problem statement, target users/personas, value proposition, core domain entities.
3. rules-extractor: finds business rules (validations, calculations, pricing/limits, permissions,
   state transitions, time/date logic, retries, idempotency, compliance). Each rule gets an ID
   (BR-001…), a plain-English statement, a location, and a confidence tag.
4. flow-mapper: traces 3–7 end-to-end real-world customer journeys (trigger → steps → systems →
   data → outcome), including unhappy paths. Output: a Mermaid flow per journey plus a one-line business purpose.
5. impact-assessor: for each journey/rule, what happens to a real customer if it fails
   (money, data, trust, legal, blocked task), plus blast radius and detectability.
6. test-strategist: risk-based test focus (see Skill: risk-scoring). Maps to the test pyramid,
   writes Given/When/Then for top risks traceable to BR-IDs, and lists what NOT to test.
7. critic: red-teams the draft. Challenges unsupported claims, downgrades confidence where
   evidence is weak, and surfaces contradictions and gaps. Nothing ships without the critic passing it,
   and that includes the HTML handoff of Stage C (it checks the page's visible text against the report).
8. html-presenter: builds the business-facing HTML handoff (Stage C) from the approved report and the
   raw findings. It presents; it never adds, upgrades or reinterprets a finding.

## A3. Skills  →  .claude/skills/<name>/SKILL.md
- evidence-citation: every claim cites `path:line` (or commit SHA) and carries one tag:
  ✅ CONFIRMED (explicit in code, tests, schema or docs) · 🟡 INFERRED (implied by naming,
  structure or data flow) · 🔴 SPECULATIVE (a guess, which must go to Open Questions).
  Never invent business context. Write in business language first, technical detail second.
- business-rule-extraction: patterns to grep for (validators, `if` guards on domain values,
  enums, status fields, money/decimal math, cron schedules, permission checks, feature flags).
- risk-scoring: Risk = Customer Impact (1–5) × Likelihood (1–5) × (1 + low-detectability bonus).
  Likelihood signals: complexity, churn (git log frequency), missing tests, external dependencies,
  concurrency, and date/money math. Score ≥15 → must test. 8–14 → test smartly. <8 → skip or monitor only.
- anti-over-testing: skip framework/library behaviour, trivial getters/DTOs, generated code,
  duplicate coverage across pyramid layers, and UI cosmetics without business impact. Prefer the
  lowest pyramid layer that can catch the risk.
- report-writing: the output template below, with length limits.
- html-handoff: the page contract, fidelity rules, visual vocabulary and verification steps of Stage C.

---------------------------------------------------------------------------------------------------
# STAGE B — RUN THE HARNESS
Run the orchestrator and write the final report to `/analysis/business-intent-report.md`.
Save intermediate subagent outputs to `/analysis/raw/`.
(`/analysis/` in this prompt means the `analysis/` folder under the repo root, never the filesystem root.)
When the critic has passed the written report, run Stage C and finish with the HTML handoff.

## OUTPUT TEMPLATE (keep the main report to about 2–3 pages)

### 1. Executive Summary (≤6 lines)
What the product is, who it serves, the problem it solves, and the single biggest customer risk.

### 2. Business Intent & Value
Problem · Personas · Value proposition · Business model hints (subscriptions, transactions, etc.),
each with a confidence tag.

### 3. Real-World Business Flows
For each key journey: purpose, a Mermaid diagram, the happy path, the top unhappy paths, and the
systems touched.

### 4. Business Rules Catalogue
Table: BR-ID | Rule (plain English) | Location | Confidence | Customer impact if broken

### 5. Customer Impact Map
Table: Flow/Rule | Failure scenario | Who is hurt and how | Severity | Detectability

### 6. Key Testing Focus (risk-based)
- Top 10 risks ranked: Risk score | BR-ID | Pyramid layer | Given/When/Then
- Explicitly NOT testing (and why), to avoid over-testing
- Coverage gaps found in existing tests

### 7. Open Questions (for Product/Business)
Every 🔴 item phrased as a question, with an owner (PO / Dev / QA / Ops) and why the answer
matters to customers.

### 8. Recommendations
Top 5 improvements (product, flow, resilience, observability, testability), each rated by
impact and effort.

### 9. Overall Customer Impact Verdict (≤5 lines)
How healthy the customer experience is today, the biggest threat, and the one thing to fix first.

### Appendix
System map, glossary, and the critic's notes.

---------------------------------------------------------------------------------------------------
# STAGE C — HTML HANDOFF (final step, after the critic has passed the written report)
Produce `analysis/business-intent-report.html`: a visual version of the approved report that a Product
Owner or business stakeholder can read without engineering knowledge. It answers three questions: what value
does this product deliver, where can customers be hurt, and what should we improve first. It presents the
approved report; it never adds, upgrades or reinterprets a finding.

## C1. Page contract
- One self-contained file: inline CSS, JS and SVG/HTML charts. No CDN, web font, image or network request, so it
  works offline (it contains security findings).
- Light and dark theme from the viewer's setting; responsive down to 320px with no sideways page scroll (wide
  tables scroll inside their own box); print-friendly; keyboard-focusable; tooltips on hover AND focus.
- Business language. No file paths or line numbers on security items and no credential values anywhere. Use plain
  names for risks, with the BR-/R- IDs shown small for cross-reference. Link to the written report and `raw/` for
  engineers (relative links; say they only work when opened from the analysis folder).
- Open with a "keep internal" notice and one paragraph on what was and was not visible (code, tests and history;
  not production settings, volumes or monitoring).

## C2. Content (in this order)
1. At a glance: three columns (Business value, Risk, Improvement), each with one number and one sentence.
2. Business value: the end-to-end flow with what the business gets at each step, who benefits (with the
   conditions under which they do), and a coverage grid of brands/markets/stores built by parsing the code's own
   store or market configuration. State that "configured in code" is not "live".
3. Where customers can be hurt: the same journey, with risk markers per step.
4. Risk: themes (plain-English consequence plus top score), a chart of how far the open questions could move the
   top scores, and the full scored-risk grid (impact x likelihood) with a table twin.
5. Who finds out: failures sorted into "told automatically / only a log line / no signal at all", and "what each
   party believes vs what is true".
6. Improvement: the Top-5 recommendations on an impact x effort grid, each mapped to the risks it reduces.
7. Open questions: every 🔴 item with owner, what it decides, and which risks it moves.
8. Confidence: the ✅/🟡/🔴 split (all tagged lines, and excluding the Open Questions rows), the limits of the
   analysis, and the critic's status.

## C3. Fidelity rules (the main failure of a visual page is losing the report's honesty)
- Take wording from the FINAL written report, not from `raw/`. Use `raw/` only for data the report does not carry
  (per-risk impact, likelihood, detectability, "what would change it"); raw files may still hold claims the critic
  corrected.
- Every visual item carries a confidence marker AND its condition ("if X is true in production (Q#)"). Three
  markers: proven in code / inferred / needs an answer. Never show a conditional harm as a fact.
- Anything the analysis added that the critic did not review (themes, recommendation-to-risk maps,
  question-to-risk maps) is labelled "our own grouping" on the page. Each source item belongs to exactly one
  group; assert it.
- Label scores honestly. "Likelihood" is test priority, not a forecast. Colour a risk by its actual score band, not
  by a derived zone. A score range shows only alternatives the analysis stated, with their scenario text; never
  draw a symmetric range that was not stated.
- Show only numbers the report states, or numbers computed from data on the page and labelled as such ("21 of the
  25 are rated silent by the analysis").
- Keep terminology consistent (for example, the order system is not "the shop").

## C4. Visual rules
- Pick the form by the data's job: a strip for a sequence, bars or dot-and-line for magnitude, a grid for a 2-D
  ranking, lanes for categories, plain large text for a single number. One axis per chart (no dual axes), thin
  marks, hairline gridlines, a legend wherever colour carries meaning, direct labels only on the marks that matter,
  text never in the series colour.
- Status colours only for status (risk band, signal level), always with a label or icon. One accent hue elsewhere.
- Every chart has a table view (collapsed `<details>`); nothing is readable only through a tooltip.
- The page is complete at rest: no content hidden behind scroll effects, no required interaction beyond hover or
  expand.

## C5. Build and verify
- Generate the page with a script, not by hand. Parse the numbers from the final report and the raw data, derive
  the counts (bands, tag split, silent share) and ASSERT them: store count, risk arithmetic, every risk in
  exactly one theme, tag counts equal to the report's. A failed assert stops the build.
- Verify once, then fix in a single pass. Render with headless Chrome at desktop width and at true phone widths
  (400, 360, 320px) plus one dark-theme sample. Headless Chrome will not lay out narrower than about 485px, so test
  phone widths inside a fixed-width iframe served over local HTTP. Check tag balance, no external URLs, no
  credential-like strings, no horizontal overflow outside scroll boxes, and that marker and label colours actually
  applied (CSS selector collisions are the usual bug).
- The critic reviews the page's visible text against the written report (conditions kept, tags, numbers,
  terminology). One revision round, same as the report.
- Do not publish or upload the page. End by offering, in one line, to publish it, and say that publishing needs
  the relative `raw/` links removed or those files published too.

---------------------------------------------------------------------------------------------------
# EXECUTION RULES
- Start by printing your plan and the list of harness files. Then build them, then run the harness.
- If the repo is large, analyse the top modules by entry-point traffic and churn first, and say what
  you skipped.
- Skip generated, vendored, minified and build-output files, plus lockfiles.
- Stage C is part of the run: it is not finished until `analysis/business-intent-report.html` exists and has passed
  the Stage C checks. The final message names both deliverables, the critic status of each, and the one-line offer
  to publish.
- End with a confidence summary: % of findings that are ✅ / 🟡 / 🔴.