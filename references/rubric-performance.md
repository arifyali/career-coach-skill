# Performance Rubric

The scoring layer. Deliberately simple so the user can read and dispute it. Three scales + two derivations.

## Evidence Strength

Applied when `/review-measures` or `/review-progress` evaluates a performance measure.

- **Weak** — zero or one supporting artifact, no corroborating feedback, OR all evidence is self-reported with no third-party signal (e.g., no 1:1 note or review doc confirms it).
- **Moderate** — two or more supporting artifacts OR one artifact plus at least one Positive/Mixed feedback entry linked to the measure.
- **Strong** — three or more supporting artifacts OR two artifacts plus explicit manager-attributed feedback (Feedback Stream row with Source: `manager` and Linked Measure matching).

Evidence Strength is a diagnostic, not a column in state. It drives whether to raise/lower the maturity call in `/review-measures`.

## Measure Maturity

The column that lives in Performance Measures Tracker. Four levels:

- **Not Yet Demonstrated** — no artifact or signal links to this measure yet. Default for freshly ingested measures.
- **Emerging** — some evidence, but limited scope OR Evidence Strength is Weak. The behavior is visible occasionally; not yet consistent.
- **Meeting** — consistent evidence at the expected scope for the user's **current** level. Evidence Strength is Moderate+.
- **Exceeding** — consistent evidence at the scope / impact of the **target** level. Evidence Strength is Strong, AND at least one piece of evidence shows the behavior at scope beyond current level.

Promotion-readiness heuristic: a measure can be marked Exceeding only when its evidence would survive challenge from a skeptical calibration committee — i.e., there is at least one artifact where Y clearly operates at target-level scope, not current-level scope.

## Trend

Derived at each `/review-progress` call. Compare the measure's **current** maturity to the maturity recorded in the **prior** Progress Snapshot row for that measure.

- **Improving** (↑) — maturity moved up at least one level since the last snapshot, OR Evidence Strength moved from Weak→Moderate or Moderate→Strong at the same maturity level.
- **Flat** (→) — maturity unchanged AND evidence strength unchanged.
- **Declining** (↓) — maturity dropped, OR Evidence Strength weakened (older evidence aging out of relevance), OR no new evidence has landed in a review window where the user is actively working on this measure.
- **Insufficient Signal** — no snapshot exists yet OR no artifacts were logged against this measure in the review window (so Trend can't be read reliably).

Trend is written to the Performance Measures Tracker's Trend column every time `/review-progress` runs.

## Priority Weight

The column indicating how much a measure matters **right now**, derived from Company Context.

- **High** — measure directly serves one of the 1-3 priorities named in `Profile.Role-to-priority mapping` AND is listed in current-cycle company or team OKRs.
- **Med** — measure serves company/team priorities indirectly, OR is in the leveling doc but not in the current OKR cycle.
- **Low** — leveling-doc requirement that isn't tied to any current-cycle priority.

Priority Weight is written by `/ingest-leveling` on first ingest and re-written by `/log-company-goals` when the user refreshes Company Context mid-cycle.

The purpose: when a user has 10+ measures, `/review-progress` prioritizes attention and the Momentum Score weights them by Priority Weight. A High-weighted measure at Emerging matters more than a Low-weighted measure at Meeting.

## Momentum Score (0-100)

A weighted-average heuristic computed at `/review-progress`. Simple on purpose. Formula:

```
weight(measure) = 3 if Priority Weight = High, 2 if Med, 1 if Low
maturity_score(measure) = 0 if Not Yet Demonstrated,
                         33 if Emerging,
                         66 if Meeting,
                         100 if Exceeding
trend_adjustment(measure) = +5 if Improving, 0 if Flat, -10 if Declining, 0 if Insufficient Signal

per_measure_score = max(0, min(100, maturity_score + trend_adjustment))
Momentum = round( sum(weight * per_measure_score) / sum(weight) )
```

Also add a **feedback floor**: if 3+ Critical feedback entries in Feedback Stream in the last 30 days are linked to High-weighted measures, cap Momentum at 55 regardless of the computed value. Rationale: formal maturity can lag live feedback by weeks; the floor forces the Trajectory Call to reflect reality.

Show the user the raw inputs to the formula in `/review-progress` output so they can see why the number is what it is. Never hide the arithmetic.

## Trajectory Call

Plain-English translation of Momentum, relative to the user's short-term Career Goal target review window.

- **Ahead** — Momentum ≥ 80 AND more than 50% of High-weighted measures are Meeting+ AND the review window is more than 90 days away. (Rare and worth celebrating; also prompts the coach to ask whether the target level is too easy.)
- **On Track** — Momentum between 65 and 79 inclusive, OR Momentum ≥ 65 with the review window less than 90 days out.
- **Lagging** — Momentum between 45 and 64 inclusive. The gap is closable but requires focused work on specific measures.
- **At Risk** — Momentum below 45, OR the feedback floor is active, OR 2+ High-weighted measures are at Not Yet Demonstrated with the review window less than 120 days away.

If the user has no short-term Career Goal set yet, output Momentum but mark Trajectory Call as `Unassessed — set a short-term goal first`.

## Review Window

The review window is the span the coach uses to assess Trend and detect stale evidence. Default: 30 days when `Review cycle timeline` is ongoing; otherwise, span from the previous `/review-progress` run through today.

An artifact "landed in the review window" if its Date is within the window. A measure is "actively worked on in the window" if at least one 1:1 Log row or Feedback Stream row in the window has it in `Linked Measures`.
