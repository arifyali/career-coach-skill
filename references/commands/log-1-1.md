# log-1-1 — Capture 1:1 notes

Record a 1:1 conversation. Prep-aware: if a Meeting Preps row exists with `Status: Scheduled` and `MGR` in its Participant IDs, offer to debrief against that prep. Otherwise, runs the standalone capture flow (unchanged from v1).

Writes to multiple sections: Meeting Log (fka 1:1 Log), Feedback Stream (for explicit feedback), Manager Background (if signals update), Stakeholder Backgrounds (if other participants were in the room), linked Measures' `Last Reviewed`, and the matched Meeting Preps row (flips `Status: Done`, appends debrief notes + per-participant Relationship Drift Flags).

Safe to run on joint-meeting preps where `MGR` is one of several participants — writes per-participant updates for everyone in the room.

## Flow

### Step 1 — Prep-aware opening check

Query Meeting Preps for rows where:
- `Status: Scheduled`
- Participant IDs contains `MGR`
- Date is within the last 14 days or in the future

If match(es) found:
- One match: "Found a prep for your 1:1 roster **[Participant Names]** from **[date]** (MTG###). Debriefing against it? (yes / standalone / discard-prep)"
- Multiple matches: list numbered with roster + date; ask which (or `none`).

### Step 2 — Branch

- `yes` → debrief-against-prep flow (Step 3+).
- `standalone` → standalone flow (Step 10). Meeting Preps row stays `Scheduled`.
- `discard-prep` → mark Meeting Preps row `Status: Cancelled` with reason, then run standalone flow.
- No match → standalone flow directly.

### Step 3 — Confirm roster (debrief-against-prep)

"In the room: **[names from prep roster]**. Correct? (If anyone was absent or added, say so now.)"

- Absent: list of IDs who didn't actually attend. Snapshots stay in prep for the record; no background updates written for them.
- Added: resolve each via FK rules; snapshot-less, so no drift check for them.

### Step 4 — Walk agenda

For each agenda item in the prep:
"On **[item]** (owner: [name]): what happened? Covered / skipped / partial? What came out of it?"

Capture one line per item.

### Step 5 — Walk anticipated questions (grouped by participant)

For each present participant, for each anticipated question in their prep subsection:
"Did [name] ask [question]? How did it land?"

Capture: asked (yes/no), response quality (strong / shaky / didn't come up).

### Step 6 — Walk questions-to-ask (grouped by participant)

For each present participant, for each question-to-ask in their prep subsection:
"Did you raise [question] with [name]? What did you learn?"

### Step 7 — Open floor

"Anything that came up we didn't prep for? Priority shifts, new asks, relationship signals?"

### Step 8 — Per-participant feedback capture

For each present participant:
- **Explicit feedback**: verbatim/paraphrase + Sentiment (Positive / Mixed / Critical) + linked measure(s) (menu from Performance Measures Tracker; multi-select; "none" valid) + linked artifact (optional `A###`).
- **Implicit signals**: tone shifts, pushback, strategy hints.
- **Follow-ups**: specific commits, deadlines, threads.

Each explicit feedback entry mirrors to Feedback Stream (dedup on Date+Source+Verbatim).

### Step 9 — Per-participant drift check

For each present participant with a snapshot in the prep, run drift checks per `references/stakeholder-relationship-types.md`:

- **Priorities drift**: snapshot top-3 vs. what the debrief surfaced. If different:
  > "You expected **[name]** to care most about **[snapshot top priority]**, but the debrief surfaced **[new priority]**. Update their background? (yes/no)"
- **Relationship Type drift** (stakeholders only): did they operate differently?
  > "This looked more like a **`Reviewer`** interaction from **[name]** than the **`Partner`** relationship on file. Reclassify? (yes/no)"
- **Open Gaps**: did any close? Any new ones open?

Accepted updates write to Manager Background (for `MGR` / `SKIP`) or Stakeholder Backgrounds (for `K###`). The Relationship Drift Flag persists in the Meeting Preps Details block for audit even after acceptance.

Then go to Step 11 — Write.

### Step 10 — Standalone flow (no prep match, or user chose standalone)

Walk the v1 sequence:
1. "Date of the 1:1?" (default today)
2. "Participant: manager or a stakeholder? (ID or name)" — If multiple participants, list them.
3. "What did you discuss? Short list."
4. Per-participant: explicit feedback (verbatim + sentiment + linked measure + linked artifact), implicit signals, follow-ups — same as Step 8.
5. If participant is `MGR` and signals suggest updates to Manager Background, ask before overwriting (confidence on confirmed fields upgrades to High).
6. Linked measures: update `Last Reviewed` for any measure referenced.

Proceed to Step 11.

### Step 11 — Write

- **Meeting Log row** (fka 1:1 Log): Date, Type (`1-1` or `group`), Participants (comma-separated attendee names), Topics, Explicit Feedback (per-participant where distinct), Implicit Signals, Follow-ups, Linked Measures, Linked Prep (`MTG###` or empty).
- **Feedback Stream rows**: one per participant per explicit feedback. Dedup on Date+Source+Verbatim.
- **Manager Background updates**: if `MGR` in the room and user accepted drift updates — Priorities, Open Gaps, Working-Style Signals; Last 1:1 Date = today.
- **Stakeholder Backgrounds updates**: for each `K###` in the room with accepted drift updates — Priorities, Open Gaps, Last Interaction = today, Relationship Last Reviewed = today (if drift fired).
- **Measures**: `Last Reviewed` = today for any referenced measure.
- **Meeting Preps row** (if matched): flip `Status: Done`, append debrief notes to Details, write per-participant Relationship Drift Flags, link `Meeting Log row ID`.
- **Session Log**: `log-1-1 | MTG### [prep or standalone], roster: [names], [N] feedback rows, [M] drift flags`.

## Coaching commentary

Follow the standard response blueprint. For 1:1 debriefs specifically:
- If Critical feedback landed on a High-Priority-Weight measure: surface the feedback-floor implication for the next `/review-progress`.
- If a pattern is forming (same feedback 3+ times across 1:1s with no measure movement): name it directly per `references/coaching-voice.md`.
- Multi-participant 1:1: if the stakeholder-side priorities drifted while manager-side didn't, that's often a signal of team-level priority movement — recommend `/log-company-goals`.

## Output Schema

```markdown
## Meeting Logged — [date] with [names]
- Type: [1-1 / group]
- Prep matched: [MTG### or "standalone"]
- Topics: [list]
- Per-participant explicit feedback:
  - [name]: [verbatim] (sentiment: [X], linked to [M###])
- Implicit signals: [1-2 lines]
- Follow-ups: [list]

## Relationship Drift Flags
- [name]: [one line description]. Updated: yes/no.

## State Updates
- Meeting Log: +1 row
- Feedback Stream: +[N] rows
- Manager Background: [field] updated (confidence: High)
- Stakeholder Backgrounds: [names] updated (if applicable)
- Measures `Last Reviewed`: [M###, M###]
- Meeting Preps MTG###: Status → Done

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
