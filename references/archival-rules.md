# Archival Rules

Keep the state file readable and within reasonable context limits for multi-year coaching engagements. Apply during `/status` at session start when the file is large, or during `/review-progress`.

## 1:1 Log Archival

When 1:1 Log exceeds 30 rows:
- Summarize older rows (older than 6 months, or older than the previous promotion/review cycle, whichever is more recent) into a `### Historical Summary` block above the recent table.
- Keep the most recent 20 rows as individual entries.
- The summary should preserve: themes raised repeatedly, major feedback inflection points, major follow-ups and whether they closed.

## Feedback Stream Archival

When Feedback Stream exceeds 40 rows:
- Summarize older-than-6-months rows into a Historical Summary block.
- Keep the most recent 30 rows + any row with Linked Artifact (artifact-linked feedback is high-signal; preserve longer).
- Historical Summary preserves: recurring themes, sentiment direction, major Critical entries and their resolution.

## Artifact Library Archival

Artifacts are longer-lived than feedback. Do not auto-archive. Instead, at year boundaries or at the user's request, offer to tag older artifacts with `Archived: [date]` in their Details block and compress the Artifact Library table rows for archived artifacts into a narrative paragraph. Keep Details blocks fully intact — these are the raw material for future interview prep.

## Progress Snapshots

Never archive. The time series IS the value — compressing it defeats the trending-over-time use case. If the table gets very long (20+ rows), reformat as a chart-ready block but keep every row.

## Performance Measures Tracker

Deprecated measures (from `/ingest-leveling` on a doc that no longer contains them) stay in the table with `(deprecated [date])` appended to the Measure field. Do not delete — preserves artifact linkage history.

## Session Log Archival

When Session Log exceeds 20 rows:
- Compress rows older than 3 months into a Historical Summary (narrative, 2-5 lines per month).
- Keep the most recent 15 rows detailed.

## Mentee deployment context

Mentees typically have shorter state files for the first year. Archival pressure usually appears after a full review cycle (6-12 months). Do not pre-emptively compress — compressing a sparse state file loses signal.
