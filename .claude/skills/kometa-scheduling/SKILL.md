---
name: kometa-scheduling
description: "Decide and write Kometa schedule, visible_home, and visible_shared values for collections and overlays so they process and appear on the right days. Use whenever a schedule, visible_home, or visible_shared value is being added or changed, when a collection isn't showing up when expected, or when designing a rotation (weekly/monthly/range) across many collection files. Encodes the ONLY valid Kometa schedule options, the schedule-vs-visibility alignment rule, and the range() pipe-vs-comma gotcha. Always cross-checks against the live Kometa nightly schedule docs."
metadata:
  version: "1.0"
  kometa_branch: nightly
---

# Kometa Scheduling

Decide and write `schedule`, `visible_home`, and `visible_shared` for collections
and overlays. Scheduling is the single most error-prone part of this config — a
wrong value silently means a collection never processes or never appears. This
skill encodes the rules that have already bitten this repo; **still confirm the
syntax against nightly source when unsure.**

## Golden rule — schedule must run at least as often as any visibility

`schedule` controls **when Kometa processes** a collection. `visible_home` /
`visible_shared` only take effect **on a run where the collection actually
processes**. If `schedule` runs less often than a `visible_*` day, the collection
won't be processed on that day and won't appear.

- ✅ Aligned: `schedule: weekly(tuesday)` + `visible_home: weekly(tuesday)`
- ✅ Broad window: `schedule: weekly(monday|tuesday|wednesday)` + `visible_home: weekly(tuesday)`
- ✅ Daily safety net: `schedule: daily` + `visible_home: weekly(monday)`
- ❌ Trap: `schedule: monthly(8)` + `visible_home: weekly(tuesday)` → skips most Tuesdays

**Prefer a scheduling window broader than the visibility day** so processing is
guaranteed to happen when visibility flips on. Collections with no `schedule`
default to `daily`.

## The ONLY valid schedule options

Do not invent options. There is no `quarterly`, `biweekly`, `bimonthly`,
`every_other_day`, etc. Valid types:

| Type | Format | Examples |
|------|--------|----------|
| Hourly | `hourly(H)` / `hourly(Start-End)` | `hourly(17)`, `hourly(17-04)` |
| Daily | `daily` | `daily` |
| Weekly | `weekly(Days)` | `weekly(sunday)`, `weekly(sunday\|tuesday)` |
| Monthly | `monthly(Day)` / `monthly(last)` | `monthly(1)`, `monthly(last)` |
| Yearly | `yearly(MM/DD)` | `yearly(01/30)` |
| Date | `date(MM/DD/YYYY)` | `date(12/25/2024)` |
| Range | `range(MM/DD-MM/DD)` | `range(12/01-12/31)` |
| Never | `never` | `never` |
| Non-existing | `non_existing` | `non_existing` |
| All (AND) | `all[Options]` | `all[weekly(sunday), hourly(17)]` |

## The range() gotcha

Multiple periods in one `range()` must be **pipe-separated**, not comma-separated:

- ❌ `range(01/01-01/31,05/01-05/31)` → "failed to parse schedule"
- ✅ `range(01/01-01/31|05/01-05/31|09/01-09/30)`
- ✅ or separate functions: `range(01/01-01/31),range(05/01-05/31)`

This affects every rotation-style schedule (actor/director/seasonal/etc.).

## End-of-month: `monthly(last)` and the `monthly(N)` change

Use `monthly(last)` to fire on the **last day of every month** regardless of length
(Feb 28/29, the 30th, or the 31st). Prefer it over guessing `monthly(28/30/31)`.

`monthly(N)` no longer falls back to the last day when day N doesn't exist that
month — it now **skips and logs a warning** (e.g. `Schedule Warning: monthly(31)
will not run this month; November does not have a 31st day`). So `monthly(31)`
silently won't run in 30-day months. (This repo's ops use `monthly(1|8|15|22)`,
which exist in every month, so they're unaffected — but anything ≥ 29 should move
to `monthly(last)`.)

## Step 1 — Confirm against source when unsure

This repo runs **nightly**. If a syntax detail is in doubt, read:

- Schedule docs: `https://kometa.wiki/en/nightly/config/schedule/`

Read it before relying on any option not in the table above. The table is the
known-good set for this config; the docs are the tie-breaker.

## Step 2 — Match this repo's rotation patterns

This config spreads work across the week to keep nightly runs fast. Mirror the
existing patterns instead of inventing new cadences:

- The runner executes Kometa **nightly**, so `daily` schedules are effectively
  the safe default and any `weekly(...)`/`monthly(...)` value must land on a day
  the runner actually runs.
- `custom/movie_subgenre/*.yml` use a shared `&sched` anchor per file (e.g.
  `weekly(sunday|monday|tuesday)`) with `visible_home`/`visible_shared` set to a
  single day inside that window (e.g. `weekly(monday)`) — the broad-window
  pattern. `config.yml` comments document which file is assigned which day.
- `config.yml` defines visibility anchors like `&daily_visibility`; reuse them.

When adding/altering a schedule, keep the window ⊇ visibility-day invariant and
keep the per-file day assignment consistent with the comments in `config.yml`.

## Step 3 — Validate

```bash
yamllint <file>.yml                    # catches quoting/indent around schedule strings
kometa --validate-file <file>.yml      # Kometa-native schema check (standalone) → validate.log
```

`kometa --validate-file` catches schema-level mistakes (bad keys/types, and many
malformed schedule strings), but it can't know your *intent* — a window narrower
than its `visible_*` day is valid YAML and valid schema, yet still wrong. Those
logic errors only surface as "failed to parse schedule" or a silently-missing
collection in `logs/meta.log` at run time. So manually re-check every change
against the two invariants:

1. Option is in the valid-options table (or confirmed in the nightly docs).
2. `schedule` window covers every `visible_home` / `visible_shared` day.

## Output

State the chosen `schedule` / `visible_*` values, show they satisfy both
invariants, and note the day assignment relative to the nightly runner. If a
requested cadence isn't expressible with a valid option (e.g. "every other
week"), say so and offer the closest valid pattern (e.g. a `range(...)` rotation
or a `weekly(...)` approximation) rather than inventing syntax.
