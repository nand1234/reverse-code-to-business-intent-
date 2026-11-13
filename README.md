# Business-intent analysis harness

A set of Claude Code subagents and skills that reverse-engineer a code repository into a business-level
picture: what problem it solves, which rules it enforces, where real customers can be hurt, and what to test
and fix first. It ends with a written report and a visual HTML version for business readers.

It is stack-neutral: nothing in the agents or skills assumes a language, framework or business domain. To use it
on a project, copy `.claude/agents`, `.claude/skills` and `.claude/prompt` into that project; see "Using it on
another repository".

## Intent

- Turn code into **evidence-backed business knowledge** for Product, QA and engineering leads, without needing
  the original authors.
- Keep every claim honest. Each claim line carries one tag and a citation: ✅ confirmed (explicit in code, tests,
  config or docs), 🟡 inferred (implied by structure), 🔴 speculative (a guess, which becomes an Open Question).
- Rank risk by customer impact, not by code smell, and say what *not* to test.
- Hand the result to the business as one page they can read without engineering knowledge.

## When to use it

- You inherit or must assess an unfamiliar service and need its business intent, rules and customer journeys.
- You are about to plan tests or a refactor and want risk-ranked focus areas traceable to rules (BR-IDs).
- You need a Product Owner / QA hand-off that separates what the code proves from what only production can answer.
- You want a repeatable review after a large change (rebrand, new market, new feature flags).

## When not to use it

- You need to know how the system **behaves in production** (load, latency, real volumes, who calls it). The
  harness reads code, tests and git history only; it does not run the app or the tests and cannot see
  production settings. It lists such unknowns as Open Questions instead.
- You need a **security audit or pentest**. It reports security-relevant findings it happens to see, with
  conditions, but it is not a substitute.
- You are **debugging one bug** or making a small change; reading the code directly is faster.
- The repo is very large and unscoped. The agents read most files outside the skip list. Scope it first, or
  expect the orchestrator to analyse only the top modules and say what it skipped.
- The result is meant to be public. The outputs contain security findings about your own systems.
- You cannot afford the run. One reference run on a ~300-file service used about 3 million subagent tokens and a
  few hours of wall time (agents run mostly in sequence). Treat that as a rough guide.

## What is here

| Path | What it is |
|---|---|
| `prompt/business.md` | The master prompt. Source of truth: it describes the harness, the stages, the report template and the HTML handoff. Running it from scratch rebuilds the harness files and then runs them. |
| `agents/orchestrator.md` | The plan: order of delegation, conflict rules, the critic gate, final assembly. Run it in the **main thread**, not as a nested agent. |
| `agents/repo-cartographer.md` | Tech stack, entry points, integrations, data stores, glossary. |
| `agents/domain-archaeologist.md` | Problem statement, personas, value, entities, from names, config and git history. |
| `agents/rules-extractor.md` | Business rules `BR-001...` with location, confidence and pinning test. |
| `agents/flow-mapper.md` | 3-7 customer journeys with Mermaid diagrams and unhappy paths. |
| `agents/impact-assessor.md` | What a customer suffers if each journey or rule fails; blast radius and detectability. |
| `agents/test-strategist.md` | Risk scores, test pyramid mapping, Given/When/Then for the top risks, what not to test. |
| `agents/critic.md` | Red-teams the draft and the HTML page: spot-checks citations, tags, numbers and wording. |
| `agents/html-presenter.md` | Builds the business-facing HTML page from the approved report. Presents only. |
| `skills/evidence-citation` | The tag and `path:line` rules every agent follows. |
| `skills/business-rule-extraction` | Search patterns and a checklist for finding rules, written as concepts to adapt to any language. |
| `skills/risk-scoring` | Risk = impact (1-5) x likelihood (1-5) x (1 + detectability bonus); bands 15 / 8. |
| `skills/anti-over-testing` | What not to test and how to pick the lowest pyramid layer. |
| `skills/report-writing` | The report template and length limits. |
| `skills/html-handoff` | Page contract, fidelity rules and verification steps for the HTML page. |

## How the stages fit together

```
01 repo-cartographer  ┐ (may run in parallel)
02 domain-archaeologist ┘
03 rules-extractor -> 04 flow-mapper -> 05 impact-assessor -> 06 test-strategist
                                                       -> 07 critic (gate on the written report)
                                                       -> 08 html-presenter -> 09 critic (HTML pass)
```

Each agent writes its own file under `analysis/raw/` and passes later agents **file paths**, not pasted content.
The orchestrator resolves conflicts (the code beats an inference; the lower confidence tag wins unless verified),
applies the critic's fixes, re-runs the critic once, and stops. One revision round per deliverable.

## How to run it with the agents and skills already here

Prerequisites: Claude Code started from the repo root (project agents and skills are loaded when a session
starts; files created mid-session may not be registered for a while, so a fresh session is the safe choice),
`git`, `python3`, and `google-chrome-stable` for the HTML checks. The target project's own runtime is not needed;
its tests are read, not run.

1. Start Claude Code in the repo root and confirm the agents are listed (for example with `/agents`). If they
   are not, restart the session, or use the `general-purpose` fallback described under Variants.
2. Move any earlier output aside so nothing is overwritten: `mv analysis analysis-$(date +%F)`.
3. Paste this prompt:

   ```
   Run the business-intent harness for this repo. You are the orchestrator: follow
   .claude/agents/orchestrator.md in the main thread (do not spawn an orchestrator agent).
   Print your plan first. Delegate to the subagents in the order of its table, passing file
   paths instead of pasted content, and write outputs to analysis/ and analysis/raw/.
   When the critic has passed the written report, run Stage C: html-presenter builds
   analysis/business-intent-report.html and the critic reviews it. Do not modify application
   code, do not commit, do not publish anything.
   ```

4. Review the outputs, in this order: `analysis/business-intent-report.md` (the written report),
   `analysis/business-intent-report.html` (open it in a browser), then `analysis/raw/` for the detail.
5. Answer the Open Questions with Product, Ops and the owning teams. They decide how severe the conditional
   findings are.

Variants:

- **Rebuild the harness from scratch and run it**: give Claude the contents of `prompt/business.md`
  (for example `@.claude/prompt/business.md`). Stage A recreates the agent and skill files, Stage B runs the
  analysis, Stage C produces the page.
- **Refresh only the HTML page** after editing the written report: run `html-presenter`, then the critic pass
  on the page. The earlier stages are not needed.
- **One agent on its own**: possible (for example only `flow-mapper`), but each agent expects the earlier
  `raw/` files it reads to exist.
- If your session does not register the agent types, spawn `general-purpose` agents with a prompt that tells
  each one to read its own `agents/<name>.md` and the skills it lists.

## Outputs

| File | Content |
|---|---|
| `analysis/business-intent-report.md` | The written report: 9 sections plus appendix, about 3-4 pages. |
| `analysis/business-intent-report.html` | The visual version for business readers. Self-contained, works offline. |
| `analysis/raw/01..06-*.md` | Each agent's full findings. |
| `analysis/raw/07-critic*.md`, `08-critic-html.md` | The critic's reviews. |

The outputs contain security findings and conditional claims. Keep them internal. The HTML links to `raw/`
with relative paths, which only work when it is opened from the `analysis` folder; publishing the page would
need those links removed or the raw files published too.

## Rules every agent follows

- Read-only on application code; never commit; never publish or upload.
- Never reproduce a credential value; cite the file and line only.
- One evidence tag per claim line, with a citation. A 🔴 claim lives in Open Questions, not in the body as fact.
- Committed config values are treated as development defaults that deployment overwrites: "configured in code"
  is not "live in production".
- A bare `git log` can be truncated in some environments (for example to the newest 50 commits). Use
  `git --no-pager log -n 5000 ...` and confirm with `git rev-list --count HEAD`.

## Using it on another repository

Copy `.claude/agents`, `.claude/skills` and `.claude/prompt` into the target project's `.claude/`, start a fresh
session there, and run as described above. No edits are required. Optional tuning before a run:

- **Seed hypotheses**: `agents/orchestrator.md` has an empty "Seed hypotheses" section. Add one-line, repo-specific
  questions there if you already suspect something; `rules-extractor` then confirms or refutes each and the critic
  re-checks the verdicts. With none, the agents say "none provided".
- **Coverage grid**: the HTML page includes a brand/market/store grid only if the repo has such a configuration to
  parse. Otherwise the presenter omits it, and the matching assert, and says so on the page.
- **Stack-specific search hints**: `skills/business-rule-extraction/SKILL.md` lists search concepts, not exact
  tokens. If your stack has distinctive idioms (annotations, DSL calls, config conventions), add them to its table.
- **Large repos**: scope the run first, as described under "When not to use it".

## Maintenance notes

- `prompt/business.md` and the files under `agents/` and `skills/` should change together. If you change the
  prompt, update the matching agent or skill.
- `.claude` is tracked in this repository. In a target project that gitignores `.claude`, remove that line or add the
  harness files explicitly. Add `analysis/` to the target's `.gitignore` if the outputs should not be committed;
  they contain security findings.
- Known limits of this design: it cannot execute tests, cannot see production, and its themes, mappings and
  scores are the analysis's own judgement; the critic reviews wording and evidence, not business priorities.
