---
name: business-rule-extraction
description: Grep patterns and a checklist for finding business rules in this PHP/Laravel 4.1 codebase (validators, guards on domain values, enums, status fields, money/date math, schedules, permissions, feature flags).
---

# Business-rule extraction

A business rule is a statement about how the business must behave that the code enforces
(or fails to enforce). Pure plumbing is not a rule. Give each rule an ID `BR-001…`, a
plain-English statement, a location, and a confidence tag (see `evidence-citation`).

## Where to look first (this repo)
`app/controllers/`, `app/validators/`, `app/models/Lifecycle*.php`, `app/models/Evoucher/**`,
`app/models/Catalog/`, `app/models/PushNotification/`, `app/config/evoucher.php`,
`app/config/voucher_authority.php`, `app/config/push_notification.php`,
`app/config/store_codes.php`, `app/config/evoucher_style.php`, `app/lang/`.

## Grep patterns (use `rg`; PHP 5.6 syntax)
| Looking for | Pattern |
|---|---|
| Validation | `Validator::make`, `'required`, `digits_between`, `'email'`, `validate` |
| Guards on domain values | `if \(.*(country|store|locale|box|status|type)`, `switch \(`, `in_array\(` |
| Status / state machine | `STEP_`, `const [A-Z_]+ =`, `status`, `state`, `->set.*Status` |
| Money / quantity | `price`, `amount`, `currency`, `exchangeable`, `round\(`, `number_format`, `bcadd` |
| Date / time | `Carbon`, `strtotime`, `date\(`, `expir`, `send_date`, `timezone`, `->diffIn` |
| Retries / idempotency | `retry`, `attempts`, `release\(`, `delete\(\)`, `Duplicate`, `already exists`, `->fire` |
| Limits | `max`, `limit`, `chunk`, `count\(.*\) [<>]`, `FVV` |
| Permissions / network | `Route::filter`, `before`, `Auth::`, `ip`, `allowed` |
| Feature flags | `enabled`, `feature`, `flag`, `Config::get\('.*(enable|active)` |
| Integrations | `Guzzle`, `curl`, `Queue::`, `S3`, `Cheetah`, `VMS`, `EAI`, `Magento`, `Gambit` |
| Compliance / privacy | `mask`, `XXX`, `confirmation_code`, `email`, `Log::`, `EvoucherLogger` |

## Per-country / per-brand rules
Config arrays keyed by country, store code or brand are rules in disguise. Read them in full and
list each distinct behaviour (not each row). Note defaults and what happens for an unknown key.

## For each rule capture
1. Who/what the rule protects (customer, money, brand, legal).
2. What happens when it is violated (reject, log, silently continue, retry).
3. Whether a test pins it (cite the test file:line, or "no test found").
4. Whether the rule lives in more than one place (duplicate = drift risk; flag it).

## Do not
- List framework behaviour as a business rule.
- Report a rule you did not locate in code — put it in Open Questions instead.
