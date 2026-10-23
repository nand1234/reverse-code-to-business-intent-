---
name: html-handoff
description: Page contract, fidelity rules, visual vocabulary and verification steps for the business-facing HTML version of the business-intent report. Use when building or checking analysis/business-intent-report.html (Stage C).
---

# HTML handoff

Audience: a Product Owner or business stakeholder. Job of the page: show what value the product delivers, where
customers can be hurt, and what to improve first, without losing the written report's honesty.
It presents the approved report. It never adds, upgrades or reinterprets a finding.

## Page contract
- One self-contained file: inline CSS, JS and SVG/HTML charts. No CDN, web font, image or network request; it must
  work offline because it contains security findings.
- Colours are tokens on `:root` (light) and redefined for dark under `prefers-color-scheme` (guarded with
  `:root:not([data-theme="light"])`) and under `:root[data-theme="dark"]`. `body` sets an explicit background.
- Responsive to 320px, a 16px side gutter, no sideways page scroll; wide tables and grids scroll inside their own
  `overflow-x: auto` box; grid children that hold text get `min-width: 0`.
- Keyboard focus is visible; tooltips open on hover AND focus; print styles hide navigation and keep colours.
- No file paths or line numbers on security items, no credential values. Plain-English risk names with the
  R-/BR- IDs shown small for cross-reference. Relative links to the written report and `raw/` for engineers, with a
  note that they only work when the file is opened from the analysis folder.
- Opens with a "keep internal" notice and one paragraph on what was visible (code, tests, history) and what was not
  (production settings, volumes, monitoring).

## Content, in this order
1. **At a glance**: three columns (value, risk, improvement), one number and one sentence each.
2. **Business value**: the end-to-end flow with "what the business gets" per step; who benefits, each with the
   condition under which they do; a brand x market coverage grid parsed from the code's store/market config, with
   layout or tier differences marked. Say "configured in code is not live".
3. **Where customers can be hurt**: the same flow as columns, each risk marker with plain text, condition,
   confidence marker and a coloured ID chip. Add a row for risks that cut across every step.
4. **Risk**: themes (plain-English consequence, top score bar with the must-test line, silent/log-only counts,
   which open questions decide it); a dot-and-line chart of how the biggest scores would change per open
   question; the full impact x likelihood grid with a table twin.
5. **Who finds out**: three lanes (told automatically / only a log line / no signal at all) and a
   "told vs true" table per party.
6. **Improvement**: Top-5 on an impact x effort grid, each card listing the risks it reduces.
7. **Open questions**: every 🔴 item as a row with owner chips, what it decides, and the risks it moves.
8. **Confidence**: stacked bars of the ✅/🟡/🔴 split, once for all tagged lines and once without the Open Questions
   rows; limits of the analysis; critic status including anything left unresolved.

## Fidelity rules
- Wording comes from the FINAL written report, not from `raw/` (raw files can hold claims the critic corrected).
  Use `raw/` only for per-risk impact, likelihood, detectability, "what would change it" and question owners.
- Confidence markers (shape plus words, never colour alone): **proven in code** (filled dot), **inferred**
  (half dot), **needs an answer** (hollow dot with "?"). A conditional harm always shows its "if ..." and the
  question number. Never state a conditional harm as a fact.
- Anything the analysis added that the critic did not review (themes, recommendation-to-risk, question-to-risk
  maps) is labelled as the analysis's own grouping. Each source item sits in exactly one group (assert it).
- "Likelihood" is test priority, not a forecast; say so beside the grid. Colour each risk chip by its actual score
  band (15+ must test, 8-14 test smartly, under 8 skip or monitor), never by an I x L zone, because the score
  also includes the detectability factor.
- A range chart plots only alternatives the analysis stated, each with its scenario text. A one-sided or
  different-scenario alternative is drawn as an alternative, never as a symmetric range.
- A number the report does not state is allowed only when computed from the data on the page and labelled
  ("21 of the 25 are rated silent by the analysis").
- Keep terminology consistent: the order system is not "the shop"; "configured" is not "live".

## Visual rules
- Form follows the data's job: strip for a sequence, bars or dot-and-line for magnitude, grid for 2-D ranking,
  lanes for categories, large plain text for a single number. One axis per chart; thin marks; hairline solid
  gridlines; a legend wherever colour carries meaning; direct labels only on the marks that matter; text never
  wears the series colour; 2px surface gaps between touching fills.
- Status colours (critical, warning, good) only for status such as score band or signal level, always with a label;
  one accent hue elsewhere. Neutral surfaces with a slight warm bias; system sans, mono for IDs.
- Every chart has a collapsed `<details>` table view; nothing is readable only through a tooltip.
- Complete at rest: no content waits on a scroll animation; no interaction is required beyond hover or expand.
- Chips and markers are at least 22-24px high so they are usable by touch.

## Build and verify
1. **Generate with a script, not by hand.** Parse the numbers from the final report and the raw data. Assert at
   least: store count and the layout split from the config; every score equals impact x likelihood x
   (1 + detectability); risks per band equal the report's; every risk is in exactly one theme; the tag counts equal
   what the report contains when its tag emoji are counted. A failed assert stops the build.
2. **Verify once, then fix in a single pass.**
   - Desktop: render with headless Chrome (`--headless=new --screenshot`), full page in one tall window, then crop
     slices. Scrolling a headless window leaves the viewport unpainted, so do not rely on `scrollTo` captures.
   - Phone: headless Chrome will not lay out narrower than about 485px, so a 400px window is a cropped 485px
     layout. Put the page in an iframe of exactly 400, 360 and 320px served over local HTTP, and measure
     `scrollWidth` against `clientWidth` and list elements that overflow outside scroll boxes.
   - Dark: render a copy with `data-theme="dark"` on `<html>` and look at one chart section.
   - Mechanical: tags are balanced, no external `http(s)` URL, no credential-like string, no `<script src>`, and a
     computed-style probe confirms the colours of markers and chips (a rule such as `.top a` can silently win over
     `.marker` when an unrelated wrapper reuses the class name; keep class names unique).
3. When you start a throwaway local server, stop it by PID. `pkill -f "<pattern>"` can match your own shell command
   and kill it.
4. The critic reviews the page's visible text against the written report. One revision round.
5. Do not publish or upload the page. Offer it in one line and say what publishing would need.
