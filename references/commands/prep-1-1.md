# prep-1-1 — Prep for your next 1:1 with your manager

Prepare for an upcoming 1:1 with the manager. Produces an agenda, anticipated questions, and user-side questions-to-ask, drawing on Manager Background + Career Goals + measures + pending feedback + (optionally) a supplied artifact. Writes a Meeting Preps row with `Status: Scheduled` that `/log-1-1` picks up at debrief time.

Supports joint 1:1s via `--with`: pull additional participants (stakeholders, skip-level) into the same prep for a 1:1 that includes other people in the room.

## Invocation

```
/prep-1-1                                                 # pure manager 1:1, no artifact
/prep-1-1 ./review-draft.pdf                              # with an artifact to review
/prep-1-1 --with K002                                     # joint 1:1 with manager + one stakeholder
/prep-1-1 --with K002,K007 ./roadmap.pdf                  # joint 1:1 with manager + two stakeholders + artifact
/prep-1-1 --with SKIP                                     # 1:1 with skip-level in the room
```

## Flow

Read `references/meeting-prep-guide.md` for the shared sequence. The prep-1-1-specific details:

### Step 1 — Resolve roster

- Seed list: `[MGR]`.
- If `--with <ids>` supplied, resolve each per the FK rules in `meeting-prep-guide.md` and append.
- Abort if Manager Background is empty: "Run `/ingest-manager` first — I need the manager profile to prep usefully."
- Echo the final roster and ask for confirmation.

### Step 2 — Capture per-participant Participant Profile Snapshots

Follow the templates in `meeting-prep-guide.md`.

### Step 3 — Meeting purpose (one question)

"What's this meeting for? One sentence."

### Step 4 — Desired outcome (one question)

"What's the one thing you want to walk away with?"

### Step 5 — Date

"When is the meeting? (default: next scheduled 1:1)"

### Step 6 — Artifact handling

- If artifact path supplied: Read the file (PDF via native Read tool, text/md directly). Produce a 2-3 line summary with confidence labels. Note which measures the artifact likely supports.
- If no artifact: skip. Prep draws only on state + snapshots.

### Step 7 — Draft the prep bundle

Per `meeting-prep-guide.md`, draft:
- Agenda (ordered per Relationship Type rules; multi-participant → highest-authority-first-for-decisions)
- Anticipated questions (grouped per participant)
- Questions-to-ask (grouped per participant)

For the pure `MGR` case, additional 1:1-specific framing:
- Lead agenda with items where the user needs manager input or sign-off, not status.
- Anticipated questions skew to manager's known priorities + any pending feedback thread not yet closed.
- Include at least one feedback-probe question-to-ask ("want to check: how am I tracking on [specific measure]?") when a 1:1 has been > 3 weeks since last.

### Step 8 — Confirm & write

- Confirm the full bundle with the user ("accept / regenerate agenda / regenerate questions / edit [section]").
- On accept: assign `MTG###`, write the Meeting Preps table row with `Status: Scheduled`, write the Details block with all snapshots + sections.
- Output the copy-paste calendar block last.

### Step 9 — Session Log

Append: `prep-1-1 | MTG###, roster: [names], artifact: [path or none]`.

## Priority Move framing

For prep commands, the Priority Move at the end of the response blueprint typically names **the single thing the user should raise if they only have 5 minutes with the manager**. Pick the item with highest downstream impact on Career Goals or the most-at-risk High-Priority-Weight measure.

## Output Schema

```markdown
## Prep Ready — MTG### — 1:1 with [names] on [date]

### What I Heard
[purpose + outcome + roster + artifact summary if any]

### What Is Working
[strong surfaces: cite A###, M###]

### Gaps To Close
[what evidence is thin going into this meeting]

### Priority Move
[the single most-leveraged thing to raise]

### Agenda (per participant)
[bulleted, named per owner]

### Anticipated questions
#### Expect [name] to ask:
- "..."
  - Prep: ...

### Questions to ask
#### Ask [name]:
- "..."
  - Why: ...

### Copy-paste calendar block
```markdown
### Agenda — [date] — 1:1 with [names]
- ...
```

**Recommended next**: `/log-1-1` after the meeting to debrief. **Alternatives**: `/log-artifact` if the meeting produced a shippable outcome.
```
