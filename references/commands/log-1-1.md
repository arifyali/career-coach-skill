# log-1-1 — Capture 1:1 notes

Record a 1:1 conversation. Writes to multiple sections: 1:1 Log, Feedback Stream (for explicit feedback), Manager Background (if signals update), and linked measures' `Last Reviewed`.

## Guided sequence — one question at a time

### Step 1 — When and with whom

- "Date of the 1:1?" (default today)
- "Participant: manager or a stakeholder?" If stakeholder, ask for the `K###` or name.

### Step 2 — Topics covered

"What did you discuss? Short list — don't retype the whole conversation."

Capture as a comma-separated or short bullet list.

### Step 3 — Explicit feedback

"Did the participant give you any explicit feedback — positive, mixed, or critical? Verbatim or paraphrase is fine."

If yes:
- Capture the verbatim/paraphrase.
- Sentiment: Positive / Mixed / Critical.
- Which measure does it map to? (show Performance Measures Tracker as menu; multiple selections ok; "none" is valid)
- Which artifact does it refer to? (optional; `A###`)

This feedback is also written to Feedback Stream automatically — the 1:1 Log and Feedback Stream rows should match on Date + Source + Verbatim to avoid duplicate work later.

### Step 4 — Implicit signals

"Any implicit signals — things they said that weren't framed as feedback but changed how you're thinking? Tone shifts, pushback on an assumption, hints about team strategy?"

Capture freeform. These are the coaching gold mine — the stuff that doesn't show up in review docs but shapes behavior.

### Step 5 — Follow-ups

"What follow-ups came out of this? Specific commits, deadlines, or threads to pick up."

### Step 6 — Manager background update (if participant is manager)

If the user mentioned anything in Steps 3-4 that would update Manager Background (new priority, new working-style signal, resolved open gap), ask:

"Does this change what we have on file for your manager? Specifically: [list the Manager Background fields that might update, pre-filled with your inference]. OK to update?"

Never overwrite without asking. Confidence on confirmed fields upgrades to High.

### Step 7 — Linked measures

If any measure IDs were tagged in Step 3 or inferred from the topics, update `Last Reviewed` on those measures to today's date.

### Step 8 — Write

- Append row to 1:1 Log.
- Append row(s) to Feedback Stream (de-dup against existing rows).
- Update Manager Background fields if confirmed.
- Update linked measures' `Last Reviewed`.
- Append Session Log row: `log-1-1 | [participant], [N] topics, [feedback or no feedback]`.

## Coaching commentary (optional)

If the 1:1 contained Critical feedback on a High-weighted measure, surface the feedback-floor implication for the next `/review-progress`. Response blueprint: What I Heard → Gaps To Close → Priority Move → Next Step.

If a pattern is forming (same feedback 3+ times across 1:1s with no measure movement), name it directly per `coaching-voice.md`.

## Output Schema

```markdown
## 1:1 Logged — [date] with [participant]
- Topics: [list]
- Explicit feedback: [verbatim] (sentiment: [X], linked to [M###])
- Implicit signals: [1-2 lines]
- Follow-ups: [list]

## State Updates
- Feedback Stream: +1 row
- Manager Background: [field name] updated to [value] (confidence: High)
- Measures `Last Reviewed`: [M###, M###]

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
