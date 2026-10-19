# XYZ Artifact Guide

Artifacts are logged in Google's XYZ format:

> **Accomplished [X] as measured by [Y], by doing [Z].**

- **X** — the outcome. What changed in the world because this work shipped. One sentence, in the past tense, candidate-specific (not "we"). Avoid activity-speak like "worked on" or "contributed to" — those are not Xs, they're inputs.
- **Y** — the measurement. **Quantified or marked `non-quant-reason`.** This is the guardrail: if you can't attach a number or an explicit reason there is no number yet, the artifact is not ready.
- **Z** — the method. The specific action, tool, decision, or sequence that produced X. Enough that a reader understands *how*, not just *what*.

**STAR conversion is explicitly downstream.** When the user is ready to prep interviews, `/export-artifacts` hands this library off to `interview-coach-skill`, which handles STAR + Earned Secret enrichment. Do not produce STAR here. Keeping the formats separate prevents double-input burden and keeps this skill focused on performance tracking.

## The Quantified-Y Guardrail

The capture flow is designed to make quantified Y the default, not the exception.

### Step 2 of `/log-artifact`: how to get to a number

Ask: "How did you measure the impact? Give me a number."

If the user has no number, offer surrogate quantifications **one at a time** until one lands or the user explicitly asks to stop:

- Volume/throughput: rows processed, requests served, tickets closed, experiments shipped
- Time: hours saved, cycle time reduced, time-to-decision cut, build time, run time
- Money: revenue impact, cost saved, forecasted ARR influenced
- Quality: AUC/precision/recall lift, false-positive rate drop, test coverage, incident rate
- Adoption: stakeholder count, DAU/WAU of a dashboard, downstream teams using the model, PRs reviewed
- Scope: # of models shipped, # of pipelines owned, # of mentees, # of stakeholders coordinated

### `non-quant-reason` escape hatch

If none of the surrogates fit, the artifact can still be logged — but only with an explicit `non-quant-reason` recorded in the Artifact Details block. Valid reasons:

- "Measurement window hasn't closed yet — results due [date]"
- "Process improvement with no yet-measured outcome; follow-up measurement planned for [date]"
- "Qualitative outcome (e.g., org change, stakeholder trust shift); evidence sources: [1:1 note, manager email, review doc]"
- "Classified / cannot share quantitative outcome in this file"

Invalid "reasons" the guardrail must reject (prompt the user to try one more time or scope down):

- "I don't remember"
- "I didn't track it"
- "It's hard to measure"

If the user cannot produce either a number or a valid `non-quant-reason` after two prompts, the capture flow stops and the coach recommends either (a) waiting until a metric is available, (b) scoping the artifact down to a measurable slice, or (c) marking it `non-quant-reason: pending measurement — follow up by [date]` and setting a Session Log reminder.

The guardrail is the point. A library of unquantified "I did a thing" entries is useless for performance tracking and useless for interviews. The friction is deliberate.

## Tagging Rules

Every artifact must link to **at least one** performance measure ID (`M###`). Artifacts not linked to any measure cannot be persisted — if a user logs something that doesn't map to any measure in the current tracker, the coach asks:

1. "Is this an area the leveling doc doesn't cover? If so, we may need to add a measure or re-ingest."
2. "Is this below-the-line / out-of-scope work? If so, log it to Coaching Notes instead of Artifact Library."

Linking to multiple measures is encouraged — a well-scoped artifact often touches 2-3.

**Competency tags** are secondary. Default to the active role profile's competency list (see `role-profiles.md`). Users can add free-form tags; keep them short and reusable (good: `experimentation`, `mentorship`, `semantic-layer`. bad: `the Q3 dashboard redesign thing`).

## Confidence Field

The Artifact Library's Confidence column exists to prevent cherry-picked overclaims from poisoning Momentum calculations. Scale:

- **High** — the user both lived the work AND can point to a measured outcome AND there is a third-party signal (manager mention, review doc, Slack thread, dashboard URL).
- **Medium** — lived work with partial measurement, OR fully measured but only self-reported ("I ran the numbers, nobody else has seen them yet").
- **Low** — in-flight, forecast-only, or heavy user interpretation of ambiguous outcomes.

`/review-progress` down-weights Low-confidence artifacts when computing a measure's Evidence Strength.

## Version History

Artifacts evolve. When the user runs `/log-artifact` with `--improve A###` or asks to "update that one about the forecasting dashboard", preserve the prior X/Y/Z and add a Version history line:

- `[date] — Updated Y with final Q3 adoption numbers (was estimated)`
- `[date] — Rewrote Z to clarify the Bayesian-threshold step after manager pushback`

Never silently overwrite. Interviews are tomorrow's problem — the user may need to recover an earlier framing.

## Session Log Hook

Every `/log-artifact` run writes a Session Log row: `Logged A###: <title> → M### maturity change: <before>→<after> if applicable`. This gives `/status` enough to produce a one-line "you've been building toward M003 — three artifacts this month" at next session.

## Minimal good example

```
A007 — Shipped incremental dbt refactor
X: Accomplished faster daily production dbt builds for the data team
Y: Measured by cutting the production run from 42 minutes to 11 minutes (73% drop), sustained over 3 weeks
Z: By doing an incremental model refactor on the three fact tables that accounted for 70% of compute, with a before/after benchmark captured in the PR description
Linked measures: M002 (Pipeline reliability), M005 (Engineering rigor)
Competency tags: dbt, pipeline-cost, performance-optimization
Confidence: High
Date: 2026-10-14
Evidence sources: PR #482, team Slack thread 2026-10-14, 1:1 note 2026-10-16 ("manager called out the win")
```

## Minimal bad example (and the fix)

```
Bad:
A008 — Worked on the churn model
X: Worked on improving the churn model
Y: It got better
Z: Various changes
```

The guardrail would reject this at step 2. The coach would walk the user back:

- "Give me a number — AUC change? Precision/recall shift? Downstream retention lift once the new scores shipped?"
- If none exist yet: "Is the measurement window still open? When does it close? Let's log this with `non-quant-reason: pending Q4 retention cohort read-out due 2027-01-10` and set a follow-up."
