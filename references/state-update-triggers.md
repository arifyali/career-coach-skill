# State Update Triggers

Write to `coaching_state.md` whenever:

- **kickoff** creates a new profile. Populates Profile, Career Goals, Company Context. Initializes empty Manager Background, Stakeholder Backgrounds, Performance Measures Tracker, Progress Snapshots, Artifact Library, 1:1 Log, Feedback Stream. Writes initial Session Log row. On re-run (Mid-Role Update), preserves Artifact Library and 1:1 Log; updates Profile and (with user confirmation) Role profile source.
- **ingest-manager** parses the LinkedIn PDF and writes Manager Background (career arc, prior roles) at `Confidence: Medium`. User-confirmed gap answers upgrade individual fields to `Confidence: High`. Sets `Last enriched: [date]`.
- **ingest-stakeholder** appends a row to Stakeholder Backgrounds and creates a corresponding Stakeholder Details block. Auto-assigns `K###`.
- **ingest-leveling** parses the leveling doc and appends rows to Performance Measures Tracker. Assigns `M###`. On re-ingest, diffs against existing rows and preserves history (deprecated rows get `(deprecated [date])` suffix, keep Supporting Artifact IDs).
- **log-company-goals** refreshes Company Context (mission, OKRs, priorities, source, last refreshed) and recomputes Priority Weight on all Performance Measures rows per the rules in `rubric-performance.md`. Appends a Session Log row.
- **log-artifact** writes a new row to Artifact Library and a Details block. Assigns `A###`. Updates linked Performance Measures' `Supporting Artifact IDs`. If this artifact pushes a measure's Evidence Strength to Moderate+ for the first time, `Current Maturity` may bump from `Emerging` to `Meeting` — surface the recommendation to the user before writing. Appends a Session Log row.
- **log-1-1** writes a 1:1 Log row. If the row contains explicit feedback, also writes a corresponding Feedback Stream row (de-duped by Date+Source+Verbatim). If the user mentions a manager priority or signal that updates Manager Background, write that too — ask before overwriting existing values. Updates linked measures' `Last Reviewed`.
- **log-feedback** writes a Feedback Stream row. If the feedback is Critical and linked to a High-weighted measure, surface the feedback floor trigger (see `rubric-performance.md`) for the next `/review-progress` call.
- **review-measures** walks the Performance Measures Tracker. May update `Current Maturity`, `Trend`, and `Last Reviewed` on each row. Does NOT write a Progress Snapshot (that's `/review-progress`).
- **review-progress** appends a new Progress Snapshot row with Momentum Score + Trajectory Call. Updates `Trend` on every measure. Shows the formula inputs in the response. Appends a Session Log row.
- **export-artifacts** writes the external file at `Profile.Interview-coach export path`. Appends a Session Log row with `Exported N artifacts to <path>`. Does NOT modify Artifact Library.
- **export-state** writes a bundle to a user-supplied path. Does NOT modify `coaching_state.md`. Appends a Session Log row.
- **import-state** overwrites `coaching_state.md` with the bundle contents AFTER explicit user confirmation. Appends a Session Log row referencing the import source.
- **status** reads state and renders the session-start summary; does not write except for appending a Session Log row for the status call itself (optional — skip if the user just opened the session without running any command).
- **Any session where the user reveals coaching-relevant personal context** (working-style preferences, review-cycle anxiety, scheduling constraints, how they handle feedback) — add to Coaching Notes.

## Mid-session saves

Write silently after any major workflow completes — not just at session end. If a long session is interrupted, no work should be lost. Only confirm saves at session end: "Session state saved. I'll pick up where we left off next time."

## Session Log cadence

Every command above that modifies state adds one row to Session Log. Keep the Key Outcomes column tight — one line.
