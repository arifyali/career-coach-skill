# prep-stakeholder — Prep for a stakeholder meeting

Prepare for an upcoming meeting with one or more stakeholders. Supports mixing in the manager or skip-level for joint meetings (quarterly reviews, roadmap syncs, cross-team roadmaps). Produces an agenda, anticipated questions, and user-side questions-to-ask, drawing on each stakeholder's Backgrounds row + measures where they're a signal source + recent feedback + (optionally) a supplied artifact. Writes a Meeting Preps row with `Status: Scheduled` that `/log-stakeholder-meeting` (or `/log-1-1`, if manager is in the room) picks up at debrief time.

## Invocation

```
/prep-stakeholder K002                                    # one-on-one stakeholder meeting
/prep-stakeholder "Priya"                                 # name-based resolution; disambiguates if multiple matches
/prep-stakeholder K002 K007                               # roadmap sync with two stakeholders
/prep-stakeholder K002 K007 --with-manager                # same + manager in the room
/prep-stakeholder K002 --with-skip ./deck.pdf             # stakeholder + skip-level + artifact
/prep-stakeholder K002 K007 K012 --with-manager ./q4.pdf  # full room with artifact
```

## Flow

Read `references/meeting-prep-guide.md` for the shared sequence. The prep-stakeholder-specific details:

### Step 1 — Resolve roster

- Parse each positional argument against Stakeholder Backgrounds:
  - Exact `K###` match first.
  - Case-insensitive substring match on Name column next. If multiple matches, show candidates (ID, Name, Role, Team, Relationship Type) and ask which.
  - No match: offer to create a lightweight Stakeholder Backgrounds row inline — ask for Name, Role, Team, Relationship Type (from the canonical enum in `references/stakeholder-relationship-types.md`), and 1-2 priorities. Assign the next `K###`.
- `--with-manager` appends `MGR` (abort if Manager Background is empty, same as `/prep-1-1`).
- `--with-skip` appends `SKIP`. If no skip-level is captured, prompt to add one before continuing.
- Echo the final roster and ask for confirmation.

### Step 2 — Capture per-participant Participant Profile Snapshots

Follow the templates in `meeting-prep-guide.md`. Each stakeholder gets a snapshot that includes their **Relationship Type** — the single most important input to how this prep is shaped.

### Step 3 — Meeting purpose (one question)

"What's this meeting for? One sentence."

Common frames for stakeholder meetings: quarterly review, roadmap alignment, blocker escalation, status update, kickoff, retro.

### Step 4 — Desired outcome (one question)

"What's the one thing you want to walk away with?"

### Step 5 — Date & duration

- "When is the meeting?" (date)
- "How long is it scheduled for? (default 30 min for 1-on-1, 45 min for group)"

Duration shapes how many agenda items fit.

### Step 6 — Artifact handling

- If artifact path supplied: Read the file. Summarize with confidence labels. Note which measures the artifact likely supports and which stakeholders' priorities it most touches.
- If no artifact: skip; prep draws on state + snapshots.

### Step 7 — Draft the prep bundle

Per `meeting-prep-guide.md`:

**Agenda**:
- Single stakeholder: order by their Relationship Type (Reviewer → evidence first, Customer → their asks first, Senior Partner → decisions needed first, etc.)
- Multi-stakeholder or mixed with MGR/SKIP: highest-authority-first for decisions, owner-first for status. Name primary participant per item.

**Anticipated questions**: group by participant. For each person, generate 2-4 questions matched to their Relationship Type + known Priorities + artifact content. Include evidence to cite (A###, M###).

**Questions-to-ask**: group by participant. Bias toward each person's Open Gaps + relationship probes if Last Interaction was > 60 days ago.

**Cross-participant signals**: if the roster has 2+ participants who are signal sources on the same measure, call out that overlap — it's typically a "both will care" agenda item.

### Step 8 — Confirm & write

- Confirm the full bundle ("accept / regenerate agenda / regenerate questions / edit [section]").
- On accept: assign `MTG###`, write Meeting Preps row (`Status: Scheduled`), write Details block.
- Output copy-paste calendar block last.

### Step 9 — Session Log

Append: `prep-stakeholder | MTG###, roster: [names], artifact: [path or none]`.

## Priority Move framing

For prep commands, the Priority Move names the single thing the user should raise if the meeting goes sideways on time. For stakeholder meetings, this usually means:
- `Customer` or `Downstream` in room → the ask the user needs to confirm you're meeting their need
- `Reviewer` in room → the strongest piece of evidence against the measure they're evaluating
- `Upstream` in room → the capacity question that unblocks the user's work
- Multi-stakeholder → the one point of alignment the user needs to leave with

## Output Schema

Same as `/prep-1-1` output schema — see `references/commands/prep-1-1.md`. Replace "1:1 with [names]" in the heading with "Meeting with [names]".
