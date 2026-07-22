# wo-ledger-data — data schema

This repo holds two distinct datasets scraped from westorange.org. They cover
different content and are **not** meant to share a naming convention or
merge into one file — noting that here since it wasn't obvious from the
filenames alone.

## `transcripts.json` — meeting transcripts

One big file, `{"meetings": [...]}`, where each entry is a single Township
Council **meeting** (agenda session, not a specific resolution):

```json
{
  "id": "2023-08-31-VT",
  "date": "2023-08-31",
  "type": "Video Transcript",
  "num_pages": 47,
  "pages": [
    { "page": 1, "text": "..." }
  ]
}
```

- `type` observed values: `Video Transcript`, `Conference Meeting`,
  `Public Meeting`, `Special Public Meeting`, `Executive Session`.
- Coverage as of this writing: **2023-08-31 through 2026-07-17** (361
  meetings). Nothing before Aug 2023.
- `pages[].text` is the transcribed/extracted text of that meeting document
  (video transcript or written minutes/agenda), page by page.

## `documents-YYYY.json` — resolution & ordinance index

One file per year, `{"documents": [...]}`, where each entry is a single
**resolution or ordinance** (not a meeting):

```json
{
  "id": "343-23",
  "type": "Resolution",
  "date": "2023-10-10",
  "title": "RESOLUTION AUTHORIZING SUBORDINATION OF MORTGAGE FOR 85 MITCHELL STREET - SEPTEMBER 2023",
  "source_url": "https://www.westorange.org/Archive.aspx?ADID=2717"
}
```

- `id` is the township's own resolution/ordinance number (`NNN-YY`, year is
  the last two digits, e.g. `343-23` = 2023). `type` is `Resolution` or
  `Ordinance`.
- `date` is the date of the Council meeting where the item appears (not
  necessarily the item's introduction date — some items get carried between
  meetings and this reflects wherever it was scraped from).
- `source_url` links to the Archive Center page for the **meeting** the
  item was pulled from, not a standalone per-resolution page — West Orange
  doesn't host individually numbered resolution PDFs. See "known gaps"
  below.
- `pages` (optional) — present only where full text has been extracted from
  the meeting's combined Packet PDF; same shape as `transcripts.json`'s
  (`[{"page": N, "text": "..."}]`, page numbers are the record's own page
  range within that meeting's packet, not the meeting's absolute page
  count). Records without `pages` are index-only (id/date/title/source_url)
  — the site falls back to title-only display for these, no code changes
  needed if `pages` gets backfilled later.

### Coverage / known gaps (as of this writing)

| Years | Status |
|---|---|
| 2012–2026 | Full index (resolutions + ordinances) + partial full text — see below |
| 2010–2011 | Exists on the site under separate Archive Center categories ("Township Council Resolutions- 2010/2011") but hasn't been scraped |

For all years, full text was extracted directly from each meeting's combined
Packet PDF via a page-anchor heuristic (each resolution/ordinance's cover
page reliably starts with `"{id} {date} RESOLUTION|ORDINANCE"`). This
catches roughly 20–40% of records per year — mainly self-contained
resolutions. The rest (exhibits, budget tables, attachments without a
standard cover page, or packets too large to safely parse in-browser) stay
index-only rather than risk mis-attributing text to the wrong record.

2012–2023 was rebuilt on 2026-07-21 using the same AgendaCenter pipeline as
2024–2026 (previously it was index-only, built via an older/rougher method
against a different source). See `EXTRACTION-STATUS-2012-2023.md` for
per-year before/after counts. Note 2012 is a genuine exception: it dropped
from 33 to 16 records because the AgendaCenter category only has 11 meeting
packets for that year (West Orange's site coverage is thin that far back);
the old 33-record file likely pulled from the separate Archive Center path,
which this rebuild deliberately did not use (see extraction status file for
detail).

Ordinance ids in the 2024+ AgendaCenter packets use a different numbering
series than resolutions (4-digit, e.g. `2834-24`, vs. resolutions' 1–3
digit `NNN-YY`) — `type` is classified by that numeric threshold (≥1000 =
Ordinance) rather than by section-header text, which proved unreliable
against the packet's own formatting.

### Duplicate ids

A small number of resolution/ordinance numbers appear referenced across more
than one meeting date in the source site (e.g. carried over from
introduction to final adoption, or a follow-up report filed under the same
number). For 2012–2023, only the first-scraped occurrence (by the site's
newest-meeting-first ordering) was kept per year, so each `id` appears at
most once within those years' files. 2024–2026 does **not** dedupe across
meetings — both occurrences are kept as separate records, since collapsing
them risked losing the follow-up context. Worth reconciling in a future
pass if consistency across all years matters.
