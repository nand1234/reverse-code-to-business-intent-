---
name: business-rule-extraction
description: Search patterns and a checklist for finding business rules in any codebase, whatever the language or framework (validators, guards on domain values, enums, status fields, money/date math, schedules, permissions, feature flags).
---

# Business-rule extraction

A business rule is a statement about how the business must behave that the code enforces
(or fails to enforce). Pure plumbing is not a rule. Give each rule an ID `BR-001…`, a
plain-English statement, a location, and a confidence tag (see `evidence-citation`).

## Where to look first
Start from the entry points and modules in `analysis/raw/01-cartographer.md`. Then read, in this order:
request handlers/controllers, validators and schemas, domain services, state-machine or lifecycle code,
models, integration clients, scheduled jobs, config tables (per-region / per-brand / per-tenant), and
translation or locale files. Test names are a rule list too: read them.

## Search patterns (use `rg`; adapt the spelling to the language and framework in use)
These are concepts, not exact tokens. Translate each into the idioms of the stack (annotations, decorators,
attributes, DSL calls, config keys).

| Looking for | What to search for |
|---|---|
| Validation | `validate`, `schema`, `required`, `regex`/`pattern`, `min`/`max`/`length`, `assert`, `guard`, `check`, validation annotations |
| Guards on domain values | `if`/`switch`/`match`/`case` over country, region, tenant, locale, status, type; membership tests against fixed lists |
| Status / state machine | `enum`, constants, `status`, `state`, `step`, `phase`, `transition`, setters that change a status |
| Money / quantity | `price`, `amount`, `currency`, `tax`, `discount`, `quantity`, rounding and decimal arithmetic |
| Date / time | `date`, `time`, `expir`, `timezone`, `duration`, `ttl`, `deadline`, `schedule`, `cron`, `age` |
| Retries / idempotency | `retry`, `attempt`, `backoff`, `idempot`, `duplicate`, `already exists`, `dedup`, `lock`, `requeue`, `ack`/`nack` |
| Limits | `max`, `limit`, `quota`, `threshold`, `batch`/`chunk`, `rate`, size or count comparisons |
| Permissions / network | `auth`, `permission`, `role`, `scope`, `allow`/`deny`, middleware, filters, guards, IP or token checks |
| Feature flags | `enabled`, `feature`, `flag`, `toggle`, `experiment`, config keys that end in `enabled`/`active` |
| Integrations | HTTP client calls, publish/consume on queues or topics, storage SDKs, mail/SMS senders, webhooks, named third-party clients |
| Compliance / privacy | `mask`, `redact`, `pii`, `consent`, `audit`, logging of request bodies, handling of passwords, tokens, personal data |

## Per-country / per-brand / per-tenant rules
Config tables keyed by country, region, tenant or brand are rules in disguise. Read them in full and
list each distinct behaviour (not each row). Note defaults and what happens for an unknown key.

## For each rule capture
1. Who/what the rule protects (customer, money, brand, legal).
2. What happens when it is violated (reject, log, silently continue, retry).
3. Whether a test pins it (cite the test file:line, or "no test found").
4. Whether the rule lives in more than one place (duplicate = drift risk; flag it).

## Do not
- List framework behaviour as a business rule.
- Report a rule you did not locate in code — put it in Open Questions instead.
- Reproduce a credential or secret value you find while searching; cite the file and line only.
