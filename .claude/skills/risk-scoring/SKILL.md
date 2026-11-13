---
name: risk-scoring
description: Score a business rule or customer journey as Customer Impact x Likelihood x (1 + low-detectability bonus) and map the score to must-test / test-smartly / skip. Use when ranking test focus.
---

# Risk scoring

**Risk = Customer Impact (1–5) × Likelihood (1–5) × (1 + low-detectability bonus)**

## Customer Impact (1–5)
| 5 | Money lost or wrongly taken, legal/compliance breach, or a purchase the customer can never use |
|---|---|
| 4 | Customer data exposed, item or service not delivered / wrong one delivered, silent loss of an order |
| 3 | Delivery delayed, wrong content/branding/locale, task blocked but recoverable by support |
| 2 | Cosmetic or degraded experience, workaround exists |
| 1 | No customer-visible effect |

## Likelihood (1–5)
Raise it for: complexity (branching, many collaborators), **missing tests** (no test pins the rule),
external dependencies (HTTP, object storage, queue, mail), concurrency / ordering, and date or money math.
Lower it for: simple, well-tested, stable code.

**Churn:** if the history is long enough to be a usable signal, measure it per file with
`git --no-pager log -n 5000 --name-only --pretty=format: -- <path> | grep -v '^$' | sort | uniq -c | sort -rn`.
(A bare `git log` can be truncated in some environments — always pass `--no-pager -n 5000` and check the
total with `git rev-list --count HEAD`; raise `-n` if the total is higher.) Use two windows: all-time, and the
last 24 months before the newest commit, because all-time counts can mostly reflect an older era of the code.
Recent churn means the rule is still moving; all-time churn means it is historically fragile. Combine with
complexity, missing tests, external dependencies and date/money logic. If the history is short or squashed,
say so and do not lean on churn.

## Detectability bonus
| Bonus | When |
|---|---|
| +0.0 | Failure is loud: the caller or customer sees it immediately, an alert fires |
| +0.5 | Failure is logged but nobody is paged / no dashboard |
| +1.0 | Failure is silent (e.g. async queue job dies after a 202, error swallowed, 200-with-error body) |

## Thresholds
- **≥ 15 → must test.**
- **8–14 → test smartly** (one cheap, targeted test at the lowest pyramid layer).
- **< 8 → skip or monitor only.**

## Output row
`Risk score | BR-ID | Pyramid layer | Given/When/Then` — show the arithmetic once, e.g. `5 × 3 × 1.5 = 22.5`.
Every top risk must trace to a BR-ID from the rules catalogue. If it cannot, go back and add the rule.
