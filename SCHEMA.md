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
- **No `pages`/body text yet** for these files — this is currently a
  title/date/number index only, not full text. A separate pass adds a
  `pages` array (same shape as `transcripts.json`'s) once the full
  resolution/ordinance text has been extracted from each meeting's combined
  Packet PDF.

### Coverage / known gaps (as of this writing)

| Years | Status |
|---|---|
| 2012–2023 | Index only (id/type/date/title/source_url), no body text |
| 2024–2026 | Not yet built |
| 2010–2011 | Exists on the site under separate Archive Center categories ("Township Council Resolutions- 2010/2011") but hasn't been scraped |

A scheduled job is in progress to backfill full `pages` text for 2012–2026
and add 2024–2026 from scratch, reusing this same file naming
(`documents-YYYY.json`) and appending a `pages` array to each existing
record rather than restructuring the schema.

### Duplicate ids

A small number of resolution numbers appear referenced across more than one
meeting date in the source site (e.g. carried over from introduction to
final adoption). Where that happened, only the first-scraped occurrence
(by the site's own newest-meeting-first ordering) was kept per year — so
each `id` appears at most once within a given year's file.
