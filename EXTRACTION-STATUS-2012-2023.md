# 2012–2023 Rebuild — Extraction Status

Run date: 2026-07-21, ~21:23–23:36 (about 2h 13m wall clock, including one
recovery from a frozen browser tab — see "Issues" below).

This was an automated overnight rebuild. It replaced the old index-only
`documents-YYYY.json` files for 2012–2023 (built via an older/rougher
method) with the same extraction pipeline used earlier tonight for
2024–2026: pulling packet PDFs from the AgendaCenter category page,
indexing every resolution/ordinance, and attaching full text wherever a
record's own cover page could be reliably located in the packet.

## Per-year results

| Year | Meetings | Old count (index-only) | New count | Ordinances | With full text |
|---|---|---|---|---|---|
| 2023 | 28 | 386 | 868 | 350 | 222 |
| 2021 | 22 | 327 | 425 | 87 | 124 |
| 2022 | 19 | 327 | 383 | 54 | 95 |
| 2019 | 22 | 313 | 385 | 67 | 107 |
| 2018 | 21 | 284 | 338 | 47 | 145 |
| 2016 | 21 | 270 | 332 | 52 | 144 |
| 2020 | 21 | 250 | 332 | 57 | 94 |
| 2017 | 23 | 224 | 322 | 63 | 152 |
| 2014 | 22 | 213 | 293 | 68 | 132 |
| 2015 | 25 | 213 | 275 | 54 | 129 |
| 2013 | 21 | 196 | 295 | 57 | 161 |
| 2012 | 11 | 33 | 16 | 2 | 12 |

**Totals: 4,264 new records vs. 3,036 old** (across all 12 years).

## Why counts mostly went up

Two things drove the increase for every year except 2012:

1. **Ordinance recovery.** The prior extraction had a bug capping ordinance
   IDs at 3 digits, which silently dropped nearly all ordinances (this
   was found and fixed during tonight's 2024–2026 build). This rebuild
   uses the corrected 4-digit-capable pattern, so ordinances that were
   previously invisible are now indexed.
2. **No cross-meeting dedup.** The old 2012–2023 files kept only the first
   occurrence of each resolution/ordinance ID per year. This rebuild
   matches the 2024–2026 convention and does **not** dedupe — an item
   that's carried from introduction to final adoption across two meetings
   appears as two records (both meetings' context is preserved). This
   accounts for most of the difference in high-volume years like 2023
   (556 unique IDs but 868 total records, 147 IDs duplicated).

## Why 2012 went down (33 → 16)

This is a genuine exception, not a bug. The AgendaCenter category used for
this rebuild only has **11 meeting packets** listed for 2012 (verified live
before the run started, per the task's source note). The old 33-record file
was apparently built from a different source — most likely the separate
Archive Center path (`Archive.aspx?AMID=42`), which the task instructions
explicitly said not to use this time since it was "superseded by" the
AgendaCenter discovery. If more complete 2012 coverage is wanted later,
that Archive Center path would need to be revisited and reconciled
separately — it wasn't in scope for this run.

## Full-text coverage

Full text (page-anchored to each record's own cover page) was recovered for
roughly 25–45% of records per year, consistent with the 2024–2026 build.
The rest are index-only (id/type/date/title/source_url) — typically
exhibits, budget tables, or attachments without a standalone cover page.

## Issues encountered

- **One frozen browser tab**, during 2017 extraction (12th of 23 meetings
  in progress). The tab stopped responding to any JavaScript, including
  trivial calls — likely a slow/pathological PDF in that batch. No data
  was lost: the tab was closed, a fresh tab was set up with the same
  pipeline, and 2017 was fully reprocessed one meeting at a time (instead
  of one fire-and-forget batch) so a repeat freeze would only cost one
  meeting, not the whole year. All 23 meetings for 2017 completed cleanly
  on the second attempt with zero errors.
- No other meeting-level errors across any of the 12 years — every packet
  URL returned a result (see `errors: []` confirmed at each checkpoint).

## What's still outstanding

- 2012 coverage is thin (11 meetings via AgendaCenter). Reconciling against
  the old Archive Center source, if that data is still wanted, is a
  separate task.
- Duplicate-ID reconciliation across 2012–2026 (carried-over items appearing
  under more than one meeting date) — noted as an outstanding item in
  `SCHEMA.md`, not addressed by this run.
- 2010–2011 remain unscraped (per `SCHEMA.md`).

## Files touched

- `documents-2012.json` through `documents-2023.json` — replaced in place.
- `documents-index.json` — counts updated for all 12 rebuilt years.
- `SCHEMA.md` — coverage table updated to reflect 2012–2026 now sharing the
  same extraction method.
- Loose `documents-YYYY-new.json` files were also left in `~/Downloads`
  (outside the `wo-ledger-data` folder) as the raw staging copies — safe to
  delete once you've spot-checked the rebuilt files.

No `git commit`/`git push` was run, per instructions — everything is staged
locally for manual review and commit.
