# review-progress — Score trajectory toward career goals

Produce a Progress Snapshot — Momentum Score and Trajectory Call. This is the "how am I trending" lens; `/review-measures` is the "what should I focus on" lens. Run `/review-progress` monthly (default) or at every major review-cycle checkpoint.

Reads: everything. The Momentum formula uses Performance Measures Tracker + Progress Snapshots history + Feedback Stream + Career Goals.

## Prerequisites

- Performance Measures Tracker must exist and be non-empty. **Exception — ramping users** (see Ramp Period in `rubric-performance.md`): an empty Tracker doesn't block the run. Skip Steps 2-4, report Momentum as `n/a — no measures yet`, and run Step 5 on the ramp checkpoints (R3 will show as due or slipping).
- `Career Goals.Short-term` should be set with a target review window; otherwise Trajectory Call is `Unassessed — set a short-term goal first`. Ramping users get a Ramp Pace call even without a target review window.

## Flow

### Step 1 — Review window

Default: 30 days (if `Review cycle timeline` is `ongoing`) OR span from previous snapshot to today.

Explicit override: `/review-progress since 2026-09-01` or `/review-progress last-quarter`.

### Step 2 — Trend per measure

For every measure in the Tracker:

1. Find the prior Progress Snapshot (if any). Compare current maturity to prior maturity.
2. Compare Evidence Strength now vs. then.
3. Apply the Trend rules from `rubric-performance.md`:
   - Maturity moved up OR Evidence Strength moved up at same level → **Improving**
   - Maturity and Evidence Strength both unchanged → **Flat**
   - Maturity dropped, OR Evidence Strength weakened, OR no new evidence in the window on an actively-worked measure → **Declining**
   - No prior snapshot OR no artifacts logged against the measure in the window → **Insufficient Signal**
4. Write the Trend to the Performance Measures Tracker Trend column.

### Step 3 — Momentum Score

Compute per the formula in `rubric-performance.md`. Show every input in the response:

```
Measure M001: Priority Weight High (w=3), Maturity Meeting (66) + Trend Improving (+5) = 71
Measure M002: Priority Weight Med  (w=2), Maturity Emerging (33) + Trend Flat (+0)      = 33
...
Weighted sum: X / weight sum: Y
Momentum = round(X/Y) = Z
```

### Step 4 — Feedback floor check

If 3+ Critical feedback entries in Feedback Stream in the last 30 days are linked to High-weighted measures, cap Momentum at 55 and announce:

"Feedback floor is active — [N] Critical feedback entries in 30 days on High-weight measures. Momentum capped at 55 regardless of math."

### Step 5 — Trajectory Call

First compute `Tenure` from `Profile.Role start date` and check whether the user is ramping (Ramp Period in `rubric-performance.md`).

**Ramping** → Ramp Pace call:

1. List every ramp checkpoint, marking its source (user success criteria or default) and status: Met / Slipping / Missed / Not yet due. Cite the state evidence for each Met (Meeting Log date, artifact ID, measure ID).
2. Apply the Ramp Pace rules: Ramping — On Pace / Ramping — Behind Pace / Ramping — At Risk.
3. Show Momentum as a baseline only, labeled as such.
4. If the first review falls before day `Ramp length + 60`, add the review-during-ramp flag.

**Not ramping** → translate Momentum to a plain-English verdict per the thresholds in `rubric-performance.md`:

- Ahead / On Track / Lagging / At Risk / Unassessed

Consider the Short-term Career Goal's `Target review window` — the thresholds shift when the window is close. On the first snapshot after the ramp ends, add the ramp-exit note.

`Pre-start — no call` if `Tenure < 0`.

### Step 6 — Snapshot note

Compose a 1-2 line note summarizing what moved since the last snapshot. Call out any major shifts: a measure jumping from Not Yet Demonstrated to Emerging, a feedback floor activating, a Momentum delta of ±10 or more.

### Step 7 — Trust Status read

Read Manager Background + Stakeholder Backgrounds. Compute:

- Distribution of Trust (them→you): count of High / Med / Low.
- Distribution of Trust (you→them): count of High / Med / Low.
- Red flags (one line each, listed in output):
  - Low-trust signal source on a High-Priority-Weight measure (person is the source on 2+ Feedback Stream rows linked to a High-weight measure M### AND their them→you is Low).
  - Any Low them→you combined with that person being `MGR` or `Reviewer` relationship type — career-progression risk going into a review window.
  - 2+ you→them Low on anyone with upcoming review-cycle involvement.

**Important**: Trust does **not** affect Momentum Score or Trajectory Call per `references/vibe-check-rubric.md` and `references/rubric-performance.md`. The read is a parallel signal, surfaced in the output but not folded into the math.

### Step 8 — Write

- Append a new Progress Snapshot row with: date, snapshot note, Measures Meeting+ count, Measures Below Target count, Momentum Score, Trajectory Call.
- Confirm all Trend column updates in Performance Measures Tracker (silent — don't ask per-measure, this is bulk).
- Append Session Log row: `review-progress | Momentum [N], Trajectory [X], delta [+/-N]`.

## Coaching commentary

Full response blueprint. At Level 5 directness, lead with the biggest signal (gap, overclaim, or inflection). At Levels 1-4, start with What Is Working.

- **What I Heard**: paraphrase of state ("You've logged 7 artifacts in the last 30 days, 4 against High-weight measures. Two Critical feedback entries on M003 from your manager.")
- **What Is Working**: measures where Trend is Improving with Strong Evidence.
- **Gaps To Close**: Declining measures, Over-assessment flags, feedback floor if active. During the ramp: slipping and missed checkpoints first; low maturity on its own is expected and is not a gap.
- **Priority Move**: the single highest-leverage next step (specific command).
- **Next Step**: the second-highest-leverage move.

## Output Schema

```markdown
## Progress Snapshot — [date]

### Trajectory: [On Track / Lagging / At Risk / Ahead / Unassessed | Ramping — On Pace / Behind Pace / At Risk | Pre-start — no call]
Momentum: **[N]/100** (delta vs. last snapshot: [+N / -N / first snapshot]) [ramping: "— baseline; thresholds apply from day [Ramp length]"]
Review window: [date] → [date]

### Ramp Checkpoints (ramping users only — day [Tenure] of [Ramp length])
| ID | Due | Checkpoint | Source | Status | Evidence |
|----|-----|------------|--------|--------|----------|
| R1 | day 14 | [checkpoint] | [default / user criteria] | [Met / Slipping / Missed / Not yet due] | [Meeting Log date, A###, M###] |
Close this week: [slipping checkpoints, or "nothing slipping"]
[If applicable: "First review lands at day [N] — ask [manager] how new hires are assessed in a partial first cycle."]

### Measure Movement
- **Improved**: M###, M###
- **Flat**: M###, M###
- **Declining**: M### — [why]
- **No signal**: M###, M###

### Momentum Math
[one-line per measure showing weight * (maturity + trend adjustment)]
Weighted sum / weight sum = Momentum
Feedback floor: [active / not active]

### Trust Status
- Them→you: [N] High, [N] Med, [N] Low
- You→them: [N] High, [N] Med, [N] Low
- Red flags: [specific people + reason, or "none"]

_Trust is a parallel signal — it does not affect Momentum. See `/vibe-check` for detail._

### Coaching Read
**What I Heard**: [paraphrase]
**What Is Working**: [1-2 lines]
**Gaps To Close**: [1-2 lines]
**Priority Move**: [specific action]
**Next Step**: [second action]

**Recommended next**: [top move]. **Alternatives**: `/log-artifact`, `/log-1-1`, `/review-measures`
```
