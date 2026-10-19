# log-artifact — Capture an accomplishment in XYZ format

Read `references/xyz-artifact-guide.md` before running. The guardrail (quantified Y or valid `non-quant-reason`) is non-negotiable.

## Guided sequence — one question at a time

### Step 1 — X (what did you do)

"In one sentence, what did you do? Past tense, candidate-specific (not 'we')."

Capture as draft X. Reject activity-speak ("worked on", "contributed to") — ask for the outcome instead.

### Step 2 — Y (how was it measured) — the guardrail

"How did you measure the impact? Give me a number."

If no number, offer surrogates one at a time until one lands or user explicitly stops (see `xyz-artifact-guide.md` for the surrogate list). If still none:

"Is this an intentionally qualitative outcome, or a measurement that hasn't landed yet? If qualitative, give me the evidence sources (1:1 note, manager email, stakeholder quote). If pending, tell me when the measurement window closes."

If the user cannot produce either a number or a valid `non-quant-reason` after two prompts, STOP and recommend:
- (a) Wait until a metric is available
- (b) Scope the artifact down to a measurable slice
- (c) Mark `non-quant-reason: pending measurement — follow up by [date]` and set a Session Log reminder

Do NOT persist the artifact if the guardrail isn't satisfied. The friction is the point.

### Step 3 — Z (how did you do it)

"How did you do it? Specific actions, tools, decisions. One paragraph."

### Step 4 — Linked measures

Show the Performance Measures Tracker as a numbered menu. "Which performance measures does this support? Pick 1 or more by ID."

If the user picks nothing or says "none fit", apply the fallback from `xyz-artifact-guide.md`:
- "Is this an area the leveling doc doesn't cover? If so, we may need to add a measure or re-ingest."
- "Or is this out-of-scope work? If so, log to Coaching Notes instead of Artifact Library."

Do not persist an artifact with zero linked measures.

### Step 5 — Competency tags

Show the active role profile's default competency tags. "Any tags for this artifact? Pick from the defaults or add new ones."

### Step 6 — Confidence

"Confidence level — High (lived + measured + third-party signal), Medium (lived + measurement partial OR fully measured but self-reported), or Low (in-flight / forecast / heavy interpretation)?"

### Step 7 — Evidence sources

"Any evidence sources I should log — dashboard URL, PR number, 1:1 note date, Slack thread, review doc?"

Capture a list.

### Step 8 — Date

Default to today. User can override.

### Step 9 — Compose and confirm

Show the final XYZ sentence: *"Accomplished [X] as measured by [Y], by doing [Z]."*

Also show the Details block that will be written:

```
A### — [title the user gave or an inferred title from X]
X: ...
Y: ...  (or non-quant-reason: ...)
Z: ...
Linked measures: ...
Competency tags: ...
Confidence: ...
Date: ...
Evidence sources: ...
```

Ask: "Does this look right? Any edits?" Apply edits, then confirm.

### Step 10 — Write

- Assign the next `A###` ID.
- Append row to Artifact Library table.
- Create the Details block.
- For each linked measure, append the new `A###` to `Supporting Artifact IDs`.
- Check whether the new artifact pushes any linked measure's Evidence Strength from Weak to Moderate, or Moderate to Strong (see `rubric-performance.md`). If yes, surface the maturity recommendation and ask whether to update `Current Maturity`.
- Append Session Log row: `log-artifact | A### logged, linked to M###[, M###]`.

## Coaching commentary (optional)

If the user wants coaching on the artifact's quality (differentiation, promotion-readiness), offer a brief take. Use the response blueprint: What I Heard → What Is Working → Gaps To Close → Priority Move → Next Step. Keep it short — one sentence per section. Full differentiation work belongs in interview-coach-skill, not here.

## Improving an existing artifact

`/log-artifact --improve A###` or "update A007 with the final Q3 numbers":

- Read the existing Details block.
- Walk Steps 2-7 with the current values as defaults.
- On write, preserve the prior X/Y/Z and add a Version history line: `[date] — <what changed>`.

## Output Schema

```markdown
## Artifact Logged: A### — [title]
*"Accomplished [X] as measured by [Y], by doing [Z]."*

- Linked measures: [list]
- Competency tags: [list]
- Confidence: [level]

## Measure Impact
- [M###]: Evidence Strength: [Weak/Moderate/Strong] (was [prior])
- [M###]: Current Maturity: [Emerging] -> suggested: [Meeting]. Accept? (yes/no)

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
