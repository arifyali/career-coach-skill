# Meeting Prep Guide

Shared logic for `/prep-1-1` and `/prep-stakeholder`. Both commands follow the sequence below. Also used by `/log-1-1` and `/log-stakeholder-meeting` for prep matching and per-participant drift checks.

## Participant FK resolution

Every Meeting Prep row stores `Participant IDs` as a comma-separated list of hard foreign keys:

- `MGR` — the current manager, resolved against Manager Background at prep/debrief time
- `K###` — a specific row in Stakeholder Backgrounds (matches the existing stakeholder ID scheme)
- `SKIP` — skip-level manager. If no skip-level is captured yet, prompt to create one before continuing

Resolution rules:
1. Exact ID match first (`K002`, `MGR`, `SKIP`).
2. Partial name match against Stakeholder Backgrounds Name column (case-insensitive substring). If multiple hits, show candidates with Relationship Type + team and ask: "Which one?"
3. No match: offer to create a lightweight Stakeholder Backgrounds row inline (name + Relationship Type from `references/stakeholder-relationship-types.md` + 1-2 priorities seed) before proceeding.
4. If `MGR` is in the list but Manager Background is empty, abort: "Run `/ingest-manager` first — I need the manager profile to prep usefully."

After resolution, echo the roster:
> Prepping for a meeting with: **[name1]** ([type1]), **[name2]** ([type2]), **[name3]** ([type3]). Correct? (accept / add [id] / drop [id])

Write both `Participant IDs` (FK list) and `Participant Names` (display copy) to the Meeting Preps row. Derive `Type`: `1-1` if list length == 1, else `group`.

## Participant Profile Snapshots

Before drafting any agenda or question, capture a per-participant snapshot block into the prep's Details. These are locked at prep time — debrief compares the live profile against them to detect drift.

### For `MGR` or `SKIP`

```markdown
#### Snapshot — [Name] (manager)
- Title / tenure / reporting line: [...]
- Top 3 Explicit Priorities: [...]
- Top 3 Open Gaps: [...]
- Working-style signals: [...]
- Directness: [1-5]
- Last 1:1 Date: [date]
```

### For `K###` (stakeholder)

```markdown
#### Snapshot — [Name] (K###, [Relationship Type])
- Role / team: [...]
- Relationship Type: [canonical enum value(s)]
- Top 3 Priorities: [...]
- Top 3 Open Gaps: [...]
- Last Interaction: [date]
- Measures where they're a signal source: [M###, M###]
```

## State inputs beyond the snapshots

Pull in (in addition to the snapshots):

- **Career Goals** short-term goal — always relevant.
- **High-Priority-Weight measures with Weak evidence** — always relevant.
- **If `MGR` or `SKIP` is in the roster**: pending Feedback Stream entries not yet linked to a Meeting Log row, Coaching Notes relevant to the manager.
- **For each `K###` in the roster**: Feedback Stream entries from them in the last 90 days, Artifact Library items tagged against measures they care about.
- **Outstanding Commitments involving the roster** — pull from the Commitments table where (Owner = me AND Owed to is a participant in the roster) OR (Owner is a participant in the roster AND Owed to = me). Both directions matter: commitments the user owes a participant become status-check agenda items; commitments a participant owes the user become probes to raise. Capture Status, Due, and `C###` for each.
- **Cross-participant signals** (multi-participant only): measures where two or more participants in the roster are signal sources. These typically surface as "both will care" agenda items.

## Agenda drafting

Markdown bullet list, copy-paste-ready for a calendar invite. Each item:

```
- [Topic] — [~mins] — owner: [name]
```

Ordering rules:

### Single-participant meeting

Order by that participant's Relationship Type per the table in `references/stakeholder-relationship-types.md`. If multiple types (e.g., `Partner, Reviewer`), lead with the type that has more at stake in this meeting (evaluative types like `Reviewer`, `Senior Partner`, `Customer` dominate when they're present).

### Multi-participant meeting

**Highest-authority-first for decisions, owner-first for status.**

1. Decisions needed from the most senior person in the room (usually `MGR` or `Senior Partner`): lead.
2. Status / updates owed to `Customer` or `Downstream`: next.
3. Working / collaboration items with `Partner` / `Peer`: last.
4. Name the primary participant per item explicitly: "Decision needed from [manager]:" / "Status update to [customer]:" / "Working session with [partner]:"

Agenda should fit the meeting duration. If the user didn't give a duration, assume 30 minutes for 1:1s and 45 minutes for group meetings; prompt if items don't fit.

### Commitments review sub-section

If any Outstanding Commitments involve the roster (from the state inputs above), add a **Commitments review** sub-section inside the agenda. One line per commitment:

```
C### [status] [due] — [commitment one-line] (owner: me / name)
```

Positioning by Relationship Type:
- If the Owed-to participant is a `Reviewer`, `Senior Partner`, or `MGR`: lead the agenda with their commitment reviews — these conversations tend to be status-first.
- If the Owed-to participant is a `Partner`, `Peer`, or `Customer`: interleave with other agenda items, grouping by participant where sensible.
- Commitments the user is Owner-waiting-on (someone else owes the user) go in the agenda as "status probes" with a tactical note: "I'll ask them where this stands."

Separate the Commitments review visually from the main agenda so the user can see at a glance what's a prior-commitment-status-check vs. new business.

## Anticipated questions

One subsection per participant. For each question, add 1-2 bullet prep notes (what to say + which artifact IDs or measures to cite).

```markdown
#### Expect [Name] to ask:
- "[question]"
  - Prep: [1 line what to say; cite A### and M###]
  - Evidence: [artifact IDs, feedback dates, specific metrics]
- "[question]"
  - Prep: [...]
```

Generate questions by crossing:
- The participant's Relationship Type (from `references/stakeholder-relationship-types.md`)
- The participant's known Priorities
- Content from the artifact (if one was supplied)
- Any pending Feedback Stream entries from this participant
- **Any Open Commitments where this participant is Owed-to**: add an anticipated question of shape "they will likely ask about the status of [C### commitment one-line]" with prep notes citing where things stand.

If the artifact exists, every anticipated question should either reference the artifact or be a question *about* the artifact's content.

## Questions to ask

One subsection per participant. These are questions the *user* should raise, biased toward closing that participant's Open Gaps + probing for drift.

```markdown
#### Ask [Name]:
- "[question]"
  - Why: closes gap on [specific Open Gap] / probes for [priority or relationship shift]
- "[question]"
  - Why: [...]
```

Prioritize:
1. Gap-filling questions on items in that participant's Open Gaps.
2. Priority probes if Company Context has shifted since the last interaction ("are quarter OKRs still shaping your team's work the same way?").
3. Relationship probes if the Last Interaction was > 60 days ago (does Relationship Type still fit?).
4. For `MGR` specifically: feedback probes on items the user has worked on since the last 1:1 — proactive asks are high-signal.
5. **Open Commitments where this participant is Owner** (something they owe the user): add a question of shape "where does [commitment one-line] stand" referencing the C### and the agreed Due. Keep it specific; vague "any update on anything" is noise.

## Copy-paste calendar block

End of prep output: a triple-fenced markdown block with **just the agenda**. Anticipated / questions-to-ask stay in conversation for the user's eyes only — those are prep notes, not meeting content.

````markdown
```markdown
### Agenda — [Date] [Meeting label]
- [item]
- [item]
- [item]
```
````

The user can copy this into a calendar invite body directly.

## Prep output structure

Follow the standard response blueprint. For prep commands, the sections map:

- **What I Heard**: coach paraphrase of the purpose + desired outcome + roster + artifact summary (if any).
- **What Is Working**: the strongest surface the user is bringing in (artifacts that cite well, closed gaps, recent feedback addressed).
- **Gaps To Close**: what the user doesn't have evidence for yet that will matter in this meeting.
- **Priority Move**: the one thing that most changes the outcome if the user raises it.
- **Next Step**: `/log-1-1` or `/log-stakeholder-meeting` after the meeting.

Then: the agenda (per participant), anticipated questions (grouped by participant), questions-to-ask (grouped by participant), and the copy-paste block.

## Writing the Meeting Preps row

Table row:

```
| MTG### | [date] | 1-1 or group | MGR, K002 | Noam Segal, Priya Chen | [purpose] | [path or empty] | M003, M007 | A012 | Scheduled |
```

Details block layout:

```markdown
#### MTG### — [date] [meeting label]
- Purpose: [full purpose]
- Desired outcome: [one line]
- Roster:
  - MGR — [name]
  - K002 — [name] ([Relationship Type])
- Artifact: [path or "none"]

[one Participant Profile Snapshot block per participant]

##### Agenda
- [...]

##### Anticipated questions (by participant)
- [per-participant subsections]

##### Questions to ask (by participant)
- [per-participant subsections]

##### Debrief notes
_(filled in by /log-1-1 or /log-stakeholder-meeting)_

##### Relationship Drift Flags
_(filled in at debrief if any)_

##### Linked Meeting Log row
_(filled in at debrief)_
```
