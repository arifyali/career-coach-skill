# Commitment Schema

Rules, status lifecycle, and extraction heuristics for the Commitments table in `coaching_state.md`. Read this before any `/commitment` invocation; also read by `/log-1-1`, `/log-stakeholder-meeting`, `/log-feedback` for their extraction hooks.

## The "accepted only" rule

A commitment is captured in the Commitments table **only when the user has explicitly accepted it**. Specifically:

- A suggestion the skill proposed is NOT a commitment until the user says they adopted it.
- A follow-up mentioned in a meeting is NOT a commitment until the user confirms "yes, I agreed to that."
- A commitment in paragraph-paste mode is NOT written until the user picks `accept` on the candidate.

Every `/commitment add` flow and every extraction hook ends with an **acceptance confirmation step** — explicit yes/no. Trust Log coupling (see below) only fires on rows written through this gate.

Rationale: commitments drive Trust (you→them). If the skill auto-promotes vague mentions, every "I should" or "we could" becomes a tracked promise and the Missed-commitment pattern floods with noise.

## Status lifecycle

```
            ┌───────────┐
            │  (new)    │
            └─────┬─────┘
                  │  acceptance confirmed
                  ▼
            ┌───────────┐    renegotiate     ┌───────────────┐
            │   Open    │────────────────────▶│ Renegotiated  │
            └─────┬─────┘                     └───────┬───────┘
                  │                                   │ (new terms = new Open row OR same row carries new Due)
         ┌────────┴────────┐                          │
   block │                 │ complete                 ▼
         ▼                 ▼                 (back to Open with updated Due)
    ┌─────────┐       ┌──────────┐
    │ Blocked │       │ Completed│
    └────┬────┘       └──────────┘
         │
         ├─ unblock → Open
         ├─ cancel → Cancelled
         └─ renegotiate → Renegotiated
```

Legal transitions:
- `Open` → `Blocked` / `Completed` / `Renegotiated` / `Cancelled`
- `Blocked` → `Open` (unblock) / `Completed` / `Renegotiated` / `Cancelled`
- `Renegotiated` → `Open` (with new Due) — same row carries the new terms; status flips back to Open
- `Completed` and `Cancelled` are terminal — row is immutable after reaching these states

Every transition appends to the Status history in the Details block with `[date] [from] → [to] — [reason / terms / evidence]`.

## Cancellation reasons

The `Cancelled` state carries a required reason tag. The one that matters for Trust Log coupling:

- `missed` — the user failed to deliver and is cancelling because the window closed. Triggers Trust Log down-event proposal.
- `superseded` — a later commitment or decision made this one moot. Neutral; no Trust Log proposal.
- `scope-changed` — stakeholder or priority changed; no longer wanted. Neutral.
- `other` — free-text reason. Neutral by default; the user can manually log a Trust Log entry via `/vibe-check` if they think the cancellation damaged trust.

## Trust Log coupling

On status flip to a Trust-affecting state, `/commitment` proposes a Trust Log entry on the **Owed-to person's** record. User confirms yes/no before writing.

### Completed → propose up-event

```
[date] you→them ↑ [prior]→[new] — kept commitment: [commitment summary] (C###)
```

Movement rules per `references/vibe-check-rubric.md`: at most one notch per event, bounded by prior state. High stays High.

### Cancelled with reason `missed` → propose down-event

```
[date] you→them ↓ [prior]→[new] — missed commitment: [reason] (C###)
```

Movement: one notch down, bounded (Low stays Low).

### Renegotiated → propose neutral event

```
[date] you→them → [prior]→[prior] — renegotiated commitment: [new terms] (C###)
```

Rationale: renegotiating *early* (before the Due date passes) is honest communication — not trust-negative, modestly trust-positive. Renegotiating *after* Due has passed is closer to a soft miss. The command flow detects which case applies:
- Renegotiated before Due → neutral default, user can override to ↑
- Renegotiated after Due passed → down-event default (`you→them ↓`), user can override

### Skip conditions

The Trust Log proposal is **skipped entirely** when:
- `Owed to` is a free-text label (e.g., `team`, `review committee`, `self`) rather than a FK (`MGR` / `K###`). No record to write to.
- Owner ≠ me. Commitments where someone else owes the user something are the user's "waiting on" list; status changes on those don't move the user's own trust.

### Confirmation UX

After each status flip prompts for its info (completion evidence, blocker, etc.), the skill emits:

> **Trust Log proposal**: write `[date] you→them [arrow] [prior]→[new] — [event description] (C###)` to [name]'s Trust Log? (yes / no / edit)

- `yes` → write to `Trust Log` sub-section in Stakeholder Details (or `Manager Trust Log` section for MGR); update the Trust column on their row/field.
- `no` → skip; status flip still writes to Commitments.
- `edit` → let the user adjust the arrow, movement, or reason text before writing.

## Paragraph-paste extraction heuristic

For `/commitment --from-paste` and the extraction hooks in `/log-1-1` / `/log-stakeholder-meeting` / `/log-feedback`. The extraction is **generous in surfacing candidates, conservative in writing** — propose anything that looks like it could be a commitment and let the user filter.

### Commitment signals (strong — propose as candidate)

- Explicit promise verbs: "I told X I'd...", "I promised...", "I committed to...", "I agreed to...", "I'll send...", "I'll have X by Y"
- Time anchors: "by Thursday", "end of week", "before our next 1:1", "end of Q4"
- Specific deliverables: "the proposal", "the Q3 doc review", "the migration plan"

### Commitment signals (weak — flag as candidate but mark "vague")

- Conditional language without acceptance: "I should probably", "I could", "I might"
- Open-ended intent: "I want to follow up with X soon"
- Hearsay: "X asked me to look into Y" without the user explicitly agreeing

Weak signals should be presented separately: "**Vague mentions (not proposed as commitments)**: [list]. Say 'promote N' if any are real accepted commitments." Doesn't auto-write — forces user to upgrade them explicitly.

### Non-commitment signals (don't propose)

- Observations and reflections: "I notice I'm getting pulled in too many directions"
- Feedback given to the user: that's a Feedback Stream candidate, surfaces separately
- Shipped accomplishments ("I shipped X"): that's an Artifact candidate, surfaces separately
- Status updates on prior commitments: these should update an existing C###, not create new ones — the extraction should scan existing Commitments and offer a status update instead

### Extraction output format

```markdown
### Candidate commitments
1. [Commitment summary] — Owed to: [FK or name string] — Due: [date or "not agreed"]
2. ...

### Vague mentions (not proposed)
- [item] — "say 'promote N' to upgrade"

### Candidate feedback entries
1. [verbatim/paraphrase] — Source: [FK or label] — Sentiment: [proposed]

### Candidate accomplishments
1. [X] — proposed Y: [number or "needs Y"] — proposed Z: [actions]

### Possible status updates on existing commitments
- C004 — [item in paste] looks like a status update on this. Flip Status to [proposed]?

For each candidate: accept / edit / reject / skip
```

User walks each candidate; accepted candidates flow to their respective capture (Commitments, Feedback Stream, Artifact Library). Session Log row summarizes totals.

## Cross-table integrity rules

- When a commitment with `Source: MTG###` is written, append `C###` to the Meeting Log row's `Follow-ups` column (the row that corresponds to the MTG debrief).
- When a commitment with `Source: Feedback Stream [date + source]` is written, append `C###` to that Feedback Stream row's `Action Taken` column.
- When a commitment is marked Completed with a Completion evidence `A###`, append `C###` to the Artifact Details block's "Evidence sources" line.
- These cross-references are one-way (from the originating row *to* the commitment) and don't require follow-up writes if the commitment is later cancelled or renegotiated — the cross-reference preserves the historical link.

## Non-moralizing stance

Parallel to `references/vibe-check-rubric.md`:

The skill does not nag the user about keeping commitments. If a user has 3 Open commitments past Due and chooses not to update them, the skill flags that pattern in `/status` and surfaces it in `/review-progress`, but doesn't try to force resolution. The user may have legitimate reasons for letting a commitment slip (scope changed, stakeholder went silent, priorities shifted) that the skill doesn't know about.

What the skill does:
1. Name the pattern directly.
2. Offer a tactical response: `/commitment C### renegotiate` is almost always better than silence; `cancel` with the right reason is better than letting it rot in Open.
3. Make the trust implication visible so the user can decide if it matters.
