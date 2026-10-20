# log-stakeholder-meeting — Debrief a stakeholder meeting

Capture a stakeholder meeting after the fact. Prep-aware: if a Meeting Preps row exists with `Status: Scheduled` and the resolved stakeholder is in its Participant IDs list, offer to debrief against that prep. Otherwise, runs a standalone capture flow.

Writes to Meeting Log, Feedback Stream, Stakeholder Backgrounds (per participant in the room), Manager Background (if `MGR` was in the room), linked Measures, and the matched Meeting Preps row (flips `Status: Done`, appends debrief notes + per-participant Relationship Drift Flags).

## Invocation

```
/log-stakeholder-meeting K002
/log-stakeholder-meeting "Priya"
/log-stakeholder-meeting                      # no target — asks "which stakeholder?"
```

## Flow

### Step 1 — Resolve target

- `K###` exact match, then partial name match against Stakeholder Backgrounds Name.
- No target supplied → list recent Stakeholder Backgrounds + any Meeting Preps rows with `Status: Scheduled` that contain `K###` participants, and ask.

### Step 2 — Prep-aware opening check

Query Meeting Preps for rows where:
- `Status: Scheduled`
- Participant IDs contains the resolved target `K###`
- Date is within the last 14 days or in the future

If match(es) found:
- One match: "Found a prep for your meeting with [Participant Names] from [date] (MTG###). Debriefing against it? (yes / standalone / discard-prep)"
- Multiple matches: list numbered, ask which (or `none`).

**Preference rule**: if two preps match (one stakeholder-only, one that includes MGR), prefer the stakeholder-only prep by default. The user can override.

### Step 3 — Branch

- `yes` → debrief-against-prep flow (Step 4+).
- `standalone` → standalone flow (Step 10). Meeting Preps row stays `Scheduled`.
- `discard-prep` → mark Meeting Preps row `Status: Cancelled` with reason, then run standalone flow.
- No match → standalone flow directly.

### Step 4 — Confirm roster

"In the room: [names from prep roster]. Correct? (If anyone was absent or added, say so now.)"

Capture:
- Absent: list of IDs who didn't actually attend. Their snapshots stay in the prep for the record, but their background rows are not updated at write time.
- Added: list of IDs who attended but weren't in the prep. Prompt to resolve each (existing `K###` or create new).

### Step 5 — Walk agenda

For each agenda item in the prep:
"On **[item]** (owner: [name]): what happened? Covered / skipped / partial? What came out of it?"

Capture one line per item.

### Step 6 — Walk anticipated questions (grouped by participant)

For each present participant, for each anticipated question in their prep subsection:
"Did [name] ask [question]? How did it land?"

Capture: asked (yes/no), response quality (strong / shaky / didn't come up).

### Step 7 — Walk questions-to-ask (grouped by participant)

For each present participant, for each question-to-ask in their prep subsection:
"Did you raise [question] with [name]? What did you learn?"

### Step 8 — Open floor

"Anything that came up we didn't prep for? Priority shifts, new asks, relationship signals?"

### Step 9 — Per-participant feedback capture

For each present participant:
- Explicit feedback: verbatim/paraphrase + Sentiment (Positive / Mixed / Critical) + linked measure(s) + linked artifact (if any).
- Implicit signals: tone shifts, pushback, strategy hints.
- Follow-ups: specific commits, deadlines, threads to pick up.

Each feedback entry writes to Feedback Stream. Each follow-up logs in the Meeting Log row's `Follow-ups` column.

Then go to Step 11 — Per-participant drift check.

### Step 10 — Standalone flow (no prep match)

Walk a lighter sequence:
1. Date + roster (resolve each participant via FK rules).
2. Topics covered.
3. Per-participant explicit feedback + implicit signals + follow-ups (same as Step 9).
4. Linked measures.

Skip Steps 5-8 (no prep to walk against). Proceed to Step 11.

### Step 11 — Per-participant drift check

For each present participant with a snapshot in the prep (debrief-against-prep flow only), run drift checks per `references/stakeholder-relationship-types.md`:

- **Priorities drift**: snapshot top-3 vs. what the debrief surfaced. If different:
  > "You expected [name] to care most about [snapshot top priority], but the debrief surfaced [new priority]. Update their background? (yes/no)"
- **Relationship Type drift** (stakeholders only): did they operate differently?
  > "This looked more like a `Reviewer` interaction from [name] than the `Partner` relationship on file. Reclassify? (yes/no)"
- **Open Gaps**: did any close? Any new ones open?

Accepted updates write to Stakeholder Backgrounds (or Manager Background for `MGR` / `SKIP`). Record the Relationship Drift Flag in the Meeting Preps Details block for audit — flags persist even after Priorities are updated, so the user can see historical shifts.

### Step 11b — Commitment extraction hook

After Follow-ups are captured per participant, prompt:

> "Any of these follow-ups are formal commitments you accepted? I can structure them in Commitments so `/vibe-check`, meeting prep, and `/status` can track them."

For each listed follow-up, present a proposed Commitment with pre-filled values (Owner=me, Owed to resolved from the meeting participant tied to the follow-up, Due inferred if the follow-up mentions a timeframe else `not agreed`, Source auto-set to today's Meeting Log row). User walks each: `accept` / `edit` / `reject`. Accepted candidates run through the acceptance confirmation step in `references/commands/commitment.md` (Add mode Step 7) and write `C###`.

Cross-reference: append each written `C###` to the Meeting Log row's `Follow-ups` column.

The hook is additive — declining all behaves identically to pre-Commitments. See `references/commitment-schema.md` for the "accepted only" rule.

### Step 12 — Write

- **Meeting Log row** (fka 1:1 Log): Date, Participants (comma-separated names of attendees), Type (`1-1` or `group`), Topics, Explicit Feedback (summary across participants, one row per participant with distinct feedback), Implicit Signals, Follow-ups (freeform text + any promoted `C###` IDs appended), Linked Measures.
- **Feedback Stream rows**: one per participant per explicit feedback captured. Dedup against existing on Date+Source+Verbatim.
- **Stakeholder Backgrounds updates**: for each present `K###` with accepted drift updates — Priorities, Open Gaps, Last Interaction = today, Relationship Last Reviewed = today.
- **Manager Background updates**: if `MGR` in the room, apply the same drift-check updates to Manager Background. Ask before overwriting.
- **Measures**: for each measure referenced in feedback, update `Last Reviewed` = today.
- **Meeting Preps row** (if matched): flip `Status: Done`, append debrief notes to Details, write per-participant Relationship Drift Flags, link `Meeting Log row ID`.
- **Session Log**: `log-stakeholder-meeting | MTG### [prep or standalone], roster: [names], [N feedback] rows, [M drift flags]`.

## Coaching commentary

Follow the standard response blueprint. For stakeholder debriefs specifically:
- If a Critical feedback landed on a High-Priority-Weight measure: surface the feedback-floor implication for the next `/review-progress`.
- If Relationship Type drift fired: name the pattern directly. If a stakeholder has drifted toward `Reviewer` behavior, that often signals a review-cycle shift the user should prep for.
- If multiple participants' priorities drifted in the same direction: name that as a Company Context signal — may be time to run `/log-company-goals`.

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
- [name]: Priorities shifted — [old] → [new]. Updated: yes/no.
- [name]: Relationship Type — [old] functioning as [new]. Reclassified: yes/no.

## State Updates
- Meeting Log: +1 row
- Feedback Stream: +[N] rows
- Stakeholder Backgrounds: [names] updated
- Manager Background: [field] updated (if applicable)
- Measures `Last Reviewed`: [M###, M###]
- Meeting Preps MTG###: Status → Done

**Recommended next**: [state-aware next step]. **Alternatives**: [list]
```
