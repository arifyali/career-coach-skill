# log-feedback — Capture inbound feedback outside of 1:1s

Captures standalone feedback from managers, stakeholders, peers, review docs, Slack, email, or recruiter signals. For feedback inside a 1:1, use `/log-1-1` (which handles both the conversation record and the feedback row).

Source is optionally attributable to a known person (`MGR` or a `K###` stakeholder). When attributed, the feedback row carries the hard FK and the row written to Stakeholder Backgrounds / Manager Background sees a side-effect update (see Step 7). When not attributable (anonymous review doc, cross-team signal, recruiter note), free-text labels are fine.

## Guided sequence — one question at a time

### Step 1 — When and from whom

- Date (default today)
- "Who's this from? Give a name, `K###`, `manager`, or a label like `peer`, `recruiter`, `review doc`, `all-hands`, `unknown`."

Resolution — mirrors the pattern in `references/meeting-prep-guide.md`:

1. `manager` or `MGR` → resolve to `MGR` (your current manager per Manager Background). Confirm the name:
   > "Attributed to **MGR — [manager name]**. Correct?"
2. `K###` → resolve to the matching Stakeholder Backgrounds row. Confirm name + Relationship Type:
   > "Attributed to **K002 — Priya Chen** (Partner). Correct?"
3. Name string → case-insensitive substring match against Stakeholder Backgrounds Name + Manager Background Name. If one hit, confirm as above. If multiple hits, list candidates (ID, Name, Role, Team, Relationship Type) and ask: "Which one?"
4. No match → offer attribution options:
   - "Create a new stakeholder for them?" (yes → inline-create K### with Name + Relationship Type from canonical enum + optional 1-2 priorities)
   - "Attribute as `peer: <name>` instead?" (yes → store free-text label with name)
   - "Keep as free-text / unknown?" (yes → user picks label)
5. Free-text labels (`peer`, `recruiter`, `review doc`, `all-hands`, `unknown`, etc.) → stored as-is in Source column; no FK.

Source column format when FK-attributed: `MGR — [name]` or `K### — [name]`. Keeps the FK visible for future lookups without needing a separate column.

### Step 2 — Channel

Channel: 1:1 / Slack / email / review / all-hands / performance committee / other

### Step 3 — The feedback itself

"What did they say? Verbatim if you have it, paraphrase if not. Short is fine — don't retype a whole thread."

### Step 4 — Sentiment

Positive / Mixed / Critical. If the user says "I don't know how to classify this", default to Mixed and note in the Verbatim/Paraphrase field.

### Step 5 — Linked measure

Show Performance Measures Tracker as a numbered menu. "Which measure does this feedback relate to? Multiple selections ok, 'none' is valid."

### Step 6 — Linked artifact

"Does this feedback refer to a specific artifact? If so, which `A###`?" Optional.

### Step 7 — Action taken

"Any action you've taken or plan to take in response? Keep it short — a line or two."

### Step 8 — Write

- Append Feedback Stream row (Date, Source, Channel, Verbatim/Paraphrase, Sentiment, Linked Measure, Linked Artifact, Action Taken).
- **If Source is `K###`**: update that stakeholder's `Last Interaction = today`. Do NOT auto-update Priorities or Open Gaps — one-sided feedback alone isn't enough signal; those updates belong in `/log-stakeholder-meeting` where both sides of the exchange are captured. If sentiment is Critical, add a Coaching Note: `[date]: Critical feedback from [K###] — consider raising at next 1:1 or stakeholder meeting.`
- **If Source is `MGR`**: do NOT bump Manager Background's `Last enriched` or create a 1:1 Log row — this isn't a conversation. If sentiment is Critical, add a Coaching Note: `[date]: Critical feedback from manager — consider raising at next /prep-1-1.`
- **If Source is free-text**: just the Feedback Stream row.
- Append Session Log row: `log-feedback | [source] [sentiment], linked to [M###]`.

## Feedback floor check

If the new row brings the count of Critical feedback entries in the last 30 days linked to High-weighted measures to 3+, surface the feedback floor trigger (see `rubric-performance.md`): "This is the third Critical signal in 30 days on a High-weight measure. Next `/review-progress` will cap Momentum at 55 until these are addressed. Worth a 1:1 agenda item."

## Coaching commentary (optional)

Keep it tight. Response blueprint: What I Heard → Gaps To Close → Priority Move. One sentence each.

Call out:
- **Patterns**: same source + same measure, 2+ times in a quarter. For FK-attributed sources, this is now reliable — the same `K###` across multiple feedback rows surfaces as a stakeholder-specific pattern.
- **Contradictions**: feedback contradicts a maturity call you have in state. Name it directly and ask whether the maturity call needs re-examination.
- **Overclaim risk**: user has an artifact at `Confidence: High` on this measure, but external feedback contradicts the self-assessment. Flag the gap.
- **Relationship drift signal**: if a `K###` with Relationship Type `Partner` is suddenly giving Critical feedback, that may be drifting toward `Reviewer` behavior — surface the possibility, let the user decide.

## Output Schema

```markdown
## Feedback Logged — [date] from [source] via [channel]
- "[verbatim/paraphrase]"
- Sentiment: [X]
- Attribution: [MGR — Name / K### — Name / free-text label]
- Linked measure(s): [M###]
- Linked artifact: [A### or none]
- Action taken: [line]

## State Updates
- Feedback Stream: +1 row
- Stakeholder Backgrounds: [name] Last Interaction updated (if K### attributed)
- Coaching Notes: [added note if Critical from MGR/K###]

## Pattern Check
- [any pattern or contradiction worth naming, or "no new patterns"]

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
