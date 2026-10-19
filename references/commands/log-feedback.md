# log-feedback — Capture inbound feedback outside of 1:1s

Captures standalone feedback from stakeholders, peers, review docs, Slack, email, or recruiter signals. For feedback inside a 1:1, use `/log-1-1` (which handles both the conversation record and the feedback row).

## Guided sequence — one question at a time

### Step 1 — When and from whom

- Date (default today)
- Source: manager / stakeholder `K###` / peer / recruiter / review doc / other
- Channel: 1:1 / Slack / email / review / all-hands / performance committee / other

### Step 2 — The feedback itself

"What did they say? Verbatim if you have it, paraphrase if not. Short is fine — don't retype a whole thread."

### Step 3 — Sentiment

Positive / Mixed / Critical. If the user says "I don't know how to classify this", default to Mixed and note in the Verbatim/Paraphrase field.

### Step 4 — Linked measure

Show Performance Measures Tracker as a numbered menu. "Which measure does this feedback relate to? Multiple selections ok, 'none' is valid."

### Step 5 — Linked artifact

"Does this feedback refer to a specific artifact? If so, which `A###`?" Optional.

### Step 6 — Action taken

"Any action you've taken or plan to take in response? Keep it short — a line or two."

### Step 7 — Write

- Append Feedback Stream row.
- Append Session Log row: `log-feedback | [source] [sentiment], linked to [M###]`.

## Feedback floor check

If the new row brings the count of Critical feedback entries in the last 30 days linked to High-weighted measures to 3+, surface the feedback floor trigger (see `rubric-performance.md`): "This is the third Critical signal in 30 days on a High-weight measure. Next `/review-progress` will cap Momentum at 55 until these are addressed. Worth a 1:1 agenda item."

## Coaching commentary (optional)

Keep it tight. Response blueprint: What I Heard → Gaps To Close → Priority Move. One sentence each.

Call out:
- Patterns: same source + same measure, 2+ times in a quarter.
- Contradictions: feedback contradicts a maturity call you have in state. Name it directly and ask whether the maturity call needs re-examination.
- Overclaim risk: user has an artifact at `Confidence: High` on this measure, but external feedback contradicts the self-assessment. Flag the gap.

## Output Schema

```markdown
## Feedback Logged — [date] from [source] via [channel]
- "[verbatim/paraphrase]"
- Sentiment: [X]
- Linked measure(s): [M###]
- Linked artifact: [A### or none]
- Action taken: [line]

## Pattern Check
- [any pattern or contradiction worth naming, or "no new patterns"]

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
