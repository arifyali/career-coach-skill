# review-measures — Walk each performance measure

For each measure in the Performance Measures Tracker, surface evidence coverage and prescribe a next move. Does NOT write a Progress Snapshot (that's `/review-progress`). This command is the "what should I focus on" lens; `/review-progress` is the "how am I trending" lens.

## Prerequisites

- `Performance Measures Tracker` must exist and be non-empty. If empty, redirect: "No measures yet. Run `/ingest-leveling <path>` first, or add role-profile seeds via `/kickoff` re-run."
- `Company Context.Role-to-priority mapping` should exist for Priority Weight to be meaningful. If missing, warn: "Priority Weights were set to default Med because Company Context is missing. Momentum math will treat all measures equally. Run `/log-company-goals` to tune."

## Flow

### Step 1 — Scan the tracker

Walk every row. For each measure, compute Evidence Strength per `rubric-performance.md`:
- Count linked artifacts.
- Count linked feedback entries (positive + mixed; critical counts as weakening evidence, not strengthening).
- Weight for manager-attributed feedback vs. other sources.
- Note any artifact at `Confidence: Low` — down-weight.

### Step 2 — For each measure, build a mini-assessment

```
M### — [measure name]
  Priority Weight: [High/Med/Low]
  Current Maturity: [X]  |  Target Maturity: [Y]
  Evidence Strength: [Weak / Moderate / Strong]
  Supporting artifacts: [list of A###] (count: N)
  Last reviewed: [date]
  Status: [On track / Needs evidence / Over-assessed / Stale]
  Recommendation: [specific next move]
```

Status is derived:
- **On track**: Current Maturity ≥ Target Maturity and Evidence Strength ≥ Moderate.
- **Needs evidence**: Current Maturity < Target Maturity and the gap is closable with 1-2 new artifacts.
- **Over-assessed**: Current Maturity is set higher than Evidence Strength supports. Flag directly: "This is marked Meeting but evidence is Weak."
- **Stale**: Last Reviewed > 60 days ago AND measure is High-weighted. Prompt for a fresh review.

### Step 3 — Update maturity calls where warranted

For any measure where Evidence Strength justifies a higher Current Maturity, ask: "M###: Evidence Strength is now Strong with 3 supporting artifacts. Promote Current Maturity from Emerging to Meeting?" (Default: ask before writing.)

For over-assessed measures, surface the dispute: "M###: Current Maturity is Meeting but Evidence Strength is Weak. I'd lower to Emerging. Agree, or do you have evidence I don't see?"

### Step 4 — Prescribe next moves

Rank the top 3 highest-leverage actions. Prefer:

1. High-weighted measures with Needs-evidence status.
2. Over-assessed measures (fix the maturity call OR add the evidence to support it).
3. Stale measures that have no recent activity.

Each recommendation names a specific `/log-artifact` theme, a `/log-1-1` topic, or a conversation to have with the manager.

### Step 5 — Write

- Update `Current Maturity` and `Last Reviewed` on measures where the user confirmed changes.
- Do NOT touch `Trend` (that's set by `/review-progress`).
- Append Session Log row: `review-measures | reviewed [N] measures, updated [M] maturity calls`.

## Output Schema

```markdown
## Measure Review — [date]

### High Priority (your anchor measures)
- M001 — [name]: [status]. [one-line recommendation]
- M003 — [name]: [status]. [one-line recommendation]
...

### Medium Priority
...

### Low Priority
[one-line summary of count + state]

## Over-Assessment Flags
- [any Current Maturity that doesn't match evidence, with specific dispute]

## Top 3 Moves
1. [specific command + reason]
2. ...
3. ...

**Recommended next**: [top-1 move]. **Alternatives**: `/log-artifact`, `/log-1-1`, `/review-progress`
```
