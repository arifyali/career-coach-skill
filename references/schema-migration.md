# Schema Migration Check

After reading `coaching_state.md` at session start, check whether it contains all sections and columns defined in the current schema (`references/coaching-state-schema.md`). Coaching state files created with earlier versions of the skill may be missing newer fields. Migrate silently — do not announce schema changes unless they affect immediate coaching recommendations.

## Section-level migrations

- **Missing `Career Goals` section**: Add the full section with empty subsections (Short-term, Long-term). Note in Coaching Notes: "[date]: Career Goals section added. Run `/kickoff` goal-only flow or edit directly."
- **Missing `Company Context` section**: Add the section header with all fields empty. Note in Coaching Notes: "[date]: Company Context section added. Run `/log-company-goals` to populate."
- **Missing `Progress Snapshots` section**: Add the empty table with the full column set. No note needed — populated by `/review-progress`.
- **Missing `Meeting Log` or `Feedback Stream`**: Add empty tables with full column sets. No note. (See the "v1 `1:1 Log` section exists but no `Meeting Log` section" rule below — it covers the rename case.)
- **Missing `Commitments` section**: Add the empty table + an empty `### Commitment Details` sub-header. No announce. Surfaces at next `/commitment` call. Do **not** auto-migrate content from existing Meeting Log `Follow-ups` or Feedback Stream `Action Taken` into Commitments — too lossy without user confirmation. The user can run `/commitment --from-paste` manually against prior follow-ups if desired.
- **Missing `Rhythms` section**: Add the empty section header between `Commitments` and `Session Log`. No announce. Populated by the next `/daily`, `/weekly`, or `/monthly` run. Older `weekly-debrief` entries without a `Recap` line are left as-is; `/monthly debrief` falls back to daily entries for those weeks.
- **Missing `Stakeholder Backgrounds`**: Add empty table + empty "Stakeholder Details" subheader. No note.
- **v1 `1:1 Log` section exists but no `Meeting Log` section**: rename the heading in-place to `## Meeting Log (fka 1:1 Log)`. Add `Type`, `Participants`, `Linked Prep` columns. Existing rows: set `Type: 1-1`, derive `Participants` from the v1 `Participant` column (`manager` → user's manager name from Manager Background; `K###` → Name lookup), `Linked Prep` empty. Preserve all existing data. No note needed — silent migration.
- **Missing `Meeting Preps` section**: Add the section header + empty table with the full column set + empty "Meeting Preps Details" subheader. No note.

## Column-level migrations in Performance Measures Tracker

- **Missing `Trend` column**: Add the column. Initialize all existing rows to `Insufficient Signal`.
- **Missing `Priority Weight` column**: Add the column. Initialize all existing rows to `Med`. Note in Coaching Notes: "[date]: Priority Weight column added — defaults to Med. Run `/log-company-goals` to recompute."
- **Missing `Supporting Artifact IDs` column**: Add the column, empty.
- **Missing `Last Reviewed` column**: Add the column, empty.

## Column-level migrations in Stakeholder Backgrounds

- **Missing `Relationship Last Reviewed` column**: Add the column, empty for all existing rows. Populated on next debrief that triggers a drift flag.
- **Missing `Trust (them→you)` or `Trust (you→them)` columns**: Add both, default every existing row to `Medium`. No announce. First-time surfacing is during next `/vibe-check` or `/people` call. See `references/vibe-check-rubric.md` for the scoring model.
- **Missing `Trust Log` sub-section** inside a Stakeholder Details block: add the sub-header empty. No announce.
- **Relationship Type values from v1 informal strings**: On first read, map to the canonical enum in `references/stakeholder-relationship-types.md`. Best-effort mapping:
  - `collaborator` → `Partner`
  - `sponsor` → `Senior Partner`
  - `blocker` → `Upstream`
  - `customer` → `Customer`
  - `peer` → `Peer`
  - Unmatched values: become `Peer (<original>)`. Add a Coaching Note: `[date]: Relationship Type values with non-canonical labels — recategorize at next /status.` /status surfaces this once.

## Column-level migrations in Artifact Library

- **Missing `Confidence` column**: Add the column. Initialize existing rows to `Medium`. Note in Coaching Notes: "[date]: Confidence column added — defaults to Medium. Re-evaluate during next `/log-artifact` cycle."
- **Missing `Competency Tags` column**: Add the column, empty.
- **Missing `Measure IDs` column**: Add the column, empty. If any artifact has no Measure IDs, flag during next `/review-measures` — artifacts must link to at least one measure (see `xyz-artifact-guide.md`).

## Field-level migrations in Manager Background

- **Missing `Trust (them→you)` or `Trust (you→them)` fields**: Add both, default to `Medium`. No announce. First-time surfacing is during next `/vibe-check` or `/people` call.
- **Missing `Manager Trust Log` sub-section** (as a `### Manager Trust Log` header beneath Manager Background): add the sub-header empty. No announce.

## Field-level migrations in Profile

- **Missing `Role profile source`**: Add the field with value `generic-data`. User can re-run `/kickoff` to pick a specific role.
- **Missing `Interview-coach export path`**: Add the field with value `../interview-coach-skill/inbox/career-artifacts.md`. User can override at next `/kickoff` or `/export-artifacts` call.
- **Missing `Mentee flag`**: Add the field with value `self`.

## When migration DOES affect immediate recommendations

If the user is running `/review-progress` and the Performance Measures Tracker is missing the `Trend` column, after migration, note in the response: "I added the Trend column to your measures — this is the first snapshot, so all Trends start at Insufficient Signal. Future snapshots will populate real trend direction."

If the user is running `/prep-stakeholder` and the stakeholder's Relationship Type was mapped to the canonical enum (or became `Peer (<original>)`), note in the response: "I remapped **[Name]**'s Relationship Type to `[canonical]` — update it with `/ingest-stakeholder` if that's not right."

Otherwise, migrate silently.
