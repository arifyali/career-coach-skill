# Archival Rules

Keep the state file readable and within reasonable context limits for multi-year coaching engagements. Apply during `/status` at session start when the file is large, or during `/review-progress`.

## Meeting Log (fka 1:1 Log) Archival

When Meeting Log exceeds 30 rows:
- Summarize older rows (older than 6 months, or older than the previous promotion/review cycle, whichever is more recent) into a `### Historical Summary` block above the recent table.
- Keep the most recent 20 rows as individual entries.
- The summary should preserve: themes raised repeatedly, major feedback inflection points, major follow-ups and whether they closed, patterns across meeting types (1-1 vs group).

## Meeting Preps Archival

Meeting Preps rows accumulate quickly. Apply at `/status` and `/review-progress`:

- **`Status: Done` rows older than 90 days**: move to a `### Archived Preps` section at the bottom of the Meeting Preps section. Keep the row in the table (shortened: ID, Date, Type, Participant Names, Status, Linked Meeting Log row). Compress the Details block to a single bullet summary: "Agenda covered X; drift flags: [list]."
- **`Status: Cancelled` rows**: archive immediately on cancellation. Keep the ID, Date, Participant Names, cancellation reason. Discard Details — cancelled preps aren't referenced again.
- **`Status: Scheduled` rows older than 30 days with no debrief**: don't archive — surface at `/status` ("MTG007 for your 1:1 with [name] is 32 days old — did the meeting happen? Run `/log-1-1` to debrief or discard-prep to clear.").

Rationale: Done preps are historical reference but not often re-read. Cancelled preps are noise. Stale Scheduled rows are a signal the user forgot a debrief — don't bury them.

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

## Rhythms Archival

Rhythm entries accumulate fastest of any section (up to ~12 per week). Apply at `/status` and `/monthly debrief`:

- Daily and weekly entries older than 60 days: compress into the Rhythms `### Historical Summary` block, 2-4 lines per month. Where that month has a confirmed `monthly-debrief` Recap, use it as the summary (plus a line on how often rhythms ran and how many were empty) — it's already user-approved. Otherwise summarize recurring intentions and risks and notable captures by ID.
- `monthly-pre` / `monthly-debrief` entries are kept for 12 months before compressing — they're small and are the long-run record.
- Keep the most recent 60 days as individual entries.
- Never delete captured IDs from the summary — they're how a reader traces back from a month's narrative to the actual records.

## Session Log Archival

When Session Log exceeds 20 rows:
- Compress rows older than 3 months into a Historical Summary (narrative, 2-5 lines per month).
- Keep the most recent 15 rows detailed.

## Mentee deployment context

Mentees typically have shorter state files for the first year. Archival pressure usually appears after a full review cycle (6-12 months). Do not pre-emptively compress — compressing a sparse state file loses signal.
