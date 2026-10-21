# coaching_state.md Schema

This is the full template for `coaching_state.md`. Use this when creating a new coaching state (during `kickoff`) or when migrating/validating an existing one. Also mirrored (empty) in `coaching_state.example.md` at the repo root — that's the file `/kickoff` copies into `coaching_state.md` on first run.

```markdown
# Career Coaching State — [Name]
Last updated: [date]

## Profile
- Role / track: [Data Scientist / Analytics Engineer / BI Engineer / Data PM / Generic Data / other]
- Role profile source: [references/role-profiles.md -> <profile key>]
- Company: [company name]
- Current level | Target next level:
- Review cycle timeline: [date of next formal review, or "ongoing"]
- Feedback directness: [1-5]
- Biggest career concern:
- Mentee flag: [self / mentee <handle>]
- Interview-coach export path: [absolute path, or `../interview-coach-skill/inbox/career-artifacts.md`, or `none`]

## Career Goals
### Short-term (6-12 months)
- Goal:
- Success criteria: [concrete, measurable — e.g., "Promoted to L4 by 2026-11 review", "shipped two end-to-end models owned solo"]
- Target review window:
- Last reviewed: [date]

### Long-term (2-5 years, optional)
- Goal: [direction / role shape — e.g., "transition to Staff IC", "move toward applied science"]
- Why this: [1-2 lines on motivation]
- Last reviewed: [date]

## Company Context
- Company mission:
- Team mission:
- Current-cycle company priorities / OKRs:
- Current-cycle team priorities / OKRs:
- Role-to-priority mapping: [which 1-3 priorities most depend on this role]
- Source: [where this came from — all-hands, team doc link, manager 1:1, etc.]
- Last refreshed: [date]

## Manager Background
- Name | Title | Tenure | Reporting line:
- Career arc (from LinkedIn PDF):
- Prior companies / roles:
- Explicit priorities (from 1:1s):
- Working-style signals: [communication cadence, decision style, feedback style]
- Open gaps / unknowns:
- Last enriched: [date]
- Confidence: [High (confirmed in 1:1) / Medium (inferred from PDF) / Low (user memory only)]
- Trust (them→you): [Low / Medium / High — default Medium. Captured via `/vibe-check`; see `references/vibe-check-rubric.md`.]
- Trust (you→them): [Low / Medium / High — default Medium.]

### Manager Trust Log
[one bullet per trust event. Format: `- [date] [direction] [↑/↓/→] [prior]→[new] — [one-line reason]`. Operational signal, never exported.]

## Stakeholder Backgrounds
| ID | Name | Role | Team | Relationship Type | Priorities | Last Interaction | Relationship Last Reviewed | Trust (them→you) | Trust (you→them) | Open Gaps |
|----|------|------|------|-------------------|------------|------------------|----------------------------|------------------|------------------|-----------|
[rows — Relationship Type uses the canonical enum defined in `references/stakeholder-relationship-types.md`: Partner / Senior Partner / Upstream / Downstream / Customer / Reviewer / Peer. Multiple types allowed, comma-separated. Relationship Last Reviewed bumps on any debrief that triggers a drift flag. Trust columns default to Medium; captured via `/vibe-check` per `references/vibe-check-rubric.md`. Operational signal, never exported.]

### Stakeholder Details
#### K001 — [Name]
- Career arc (from LinkedIn PDF):
- Explicit priorities:
- Working-style signals:
- Open gaps:
- Enrichment history: [date — what changed]
- Confidence: [High / Medium / Low]

##### Trust Log
[one bullet per trust event. Format: `- [date] [direction] [↑/↓/→] [prior]→[new] — [one-line reason]`. Operational signal, never exported.]

[repeat Details block per stakeholder]

## Performance Measures Tracker
| ID | Measure | Source | Current Maturity | Target Maturity | Trend | Priority Weight | Supporting Artifact IDs | Last Reviewed |
|----|---------|--------|------------------|-----------------|-------|-----------------|-------------------------|---------------|
[rows — Maturity: Not Yet Demonstrated / Emerging / Meeting / Exceeding. Trend: Improving / Flat / Declining / Insufficient Signal. Priority Weight: High / Med / Low — derived from Company Context role-to-priority mapping. Source: leveling doc section name or "role-profile seed" when no doc exists yet.]

## Progress Snapshots
| Date | Snapshot Note | Measures Meeting+ | Measures Below Target | Momentum Score | Trajectory Call |
|------|---------------|-------------------|-----------------------|----------------|-----------------|
[rows — Momentum Score: 0-100 (see references/rubric-performance.md). Trajectory Call: On Track / Lagging / At Risk / Ahead. One row per `/review-progress` call. Keeps a time series — do not overwrite.]

## Artifact Library
| ID | Title | Date | X (Accomplished) | Y (Measured by) | Z (By doing) | Measure IDs | Competency Tags | Confidence |
|----|-------|------|------------------|-----------------|--------------|-------------|-----------------|------------|
[rows — compact index. Full details below. Y must be quantified unless the Details block carries a `non-quant-reason`. Confidence: High (lived + measured) / Medium (lived, measurement partial) / Low (early-stage or self-reported only).]

### Artifact Details
#### A001 — [Title]
- X: [Accomplished — one sentence]
- Y: [Measured by — quantified, OR `non-quant-reason: <why no number yet>`]
- Z: [By doing — the specific action/approach/decisions]
- Linked measures: [M001, M003]
- Competency tags: [from role-profile active tags, plus any user-added]
- Evidence sources: [1:1 note dates, dashboard URL, system data, review doc]
- Date:
- Version history: [date — what changed]

[repeat per artifact]

## Meeting Preps
| ID | Date | Type | Participant IDs | Participant Names | Purpose | Artifact Path | Linked Measures | Linked Artifacts | Status |
|----|------|------|-----------------|-------------------|---------|---------------|-----------------|------------------|--------|
[rows — ID: `MTG###`. Type (derived): `1-1` if one participant, `group` if 2+. Participant IDs: comma-separated hard FKs (`MGR` / `K###` / `SKIP`). Participant Names: parallel display copy. Status: Scheduled / Done / Cancelled. Full field definitions and prep/debrief flows live in `references/meeting-prep-guide.md`.]

### Meeting Preps Details
#### MTG001 — [date] [meeting label]
- Purpose:
- Desired outcome:
- Roster: [per-participant: ID — Name (Relationship Type for stakeholders)]
- Artifact: [path or "none"]

##### Snapshots
[one Participant Profile Snapshot block per participant — see `references/meeting-prep-guide.md`]

##### Agenda
[bulleted, copy-paste-ready, each item names its primary owner]

##### Anticipated questions (by participant)
[per-participant subsections]

##### Questions to ask (by participant)
[per-participant subsections]

##### Debrief notes
_(filled in by /log-1-1 or /log-stakeholder-meeting)_

##### Relationship Drift Flags
_(filled in at debrief if any)_

##### Linked Meeting Log row
_(filled in at debrief)_

[repeat per meeting prep]

## Meeting Log (fka 1:1 Log)
| Date | Type | Participants | Topics | Explicit Feedback | Implicit Signals | Follow-ups | Linked Measures | Linked Prep |
|------|------|--------------|--------|-------------------|------------------|------------|-----------------|-------------|
[rows — Type: `1-1` / `group`. Participants: comma-separated names of attendees (not FKs — this is the human-readable record). Linked Prep: `MTG###` or empty if standalone. Explicit Feedback is also mirrored into Feedback Stream automatically. v1 state files that have a `1:1 Log` section are read as Meeting Log with Type inferred (`1-1`) and Linked Prep empty — see `references/schema-migration.md`.]

## Feedback Stream
| Date | Source | Channel | Verbatim/Paraphrase | Sentiment | Linked Measure | Linked Artifact | Action Taken |
|------|--------|---------|---------------------|-----------|----------------|-----------------|--------------|
[rows — Source: FK-attributed when possible (`MGR — [name]` or `K### — [name]`) or free-text label (`peer: alex` / `recruiter` / `review doc` / `all-hands` / `unknown`). Attribution flow lives in `references/commands/log-feedback.md`. Channel: 1:1 / Slack / email / review / all-hands. Sentiment: Positive / Mixed / Critical. Action Taken column may include `C###` references to Commitments if `/commitment` was used to formalize an action out of this feedback.]

## Commitments
| ID | Commitment | Owner | Owed to | Due | Status | Source | Linked Measures | Last Updated |
|----|------------|-------|---------|-----|--------|--------|-----------------|---------------|
[rows — ID: `C###`. Owner: `me` or FK (`MGR` / `K###`). Owed to: FK or free-text label (`team`, `review committee`, `self`). Due: ISO date or `not agreed`. Status: Open / Blocked / Completed / Renegotiated / Cancelled — lifecycle rules in `references/commitment-schema.md`. Source: `MTG###` / Meeting Log date / Feedback Stream date+source / `standalone`. Captured only after an explicit acceptance confirmation; see `references/commands/commitment.md`.]

### Commitment Details
#### C001 — [commitment one-line]
- Description: [longer form if needed]
- Status history:
  - [date] (new) → Open — [reason / source]
  - [date] Open → [new state] — [reason / terms / evidence]
- Blocker: [filled on Status: Blocked]
- Completion evidence: [`A###` or short note; filled on Status: Completed]
- Renegotiation terms: [filled on Status: Renegotiated]
- Cancellation reason: [filled on Status: Cancelled; one of `missed` / `superseded` / `scope-changed` / `other: <text>`]

[repeat Details block per commitment]

## Rhythms
### Historical Summary (when entries are older than 60 days, summarize here)
[Brief narrative per month: recurring intentions/risks, empty-entry pattern, notable captures.]

### [YYYY-MM-DD]
#### daily-pre ([HH:MM])
- Intention: [one line]
- Biggest risk: [one line]
- Raise with: [one line]
- Captures: [e.g., "1 commitment (C004)" or "none"]

#### daily-debrief ([HH:MM])
- Shipped: [one line]
- Unresolved: [one line]
- Surprises: [one line]
- Captures: [summary], [N] commitment flips

#### weekly-pre ([day] [HH:MM])
- Outcome / Biggest risk / Commitments to lock / Prep queue / Captures

#### weekly-debrief ([day] [HH:MM])
- Recap: [confirmed roll-up of the week's daily entries, 3-5 lines]
- Progress Snapshot / Headline / Career-goal movement / Shifts / Commitment flips / Trust review / Captures

#### monthly-pre ([HH:MM])
- Focus / Evidence by month-end / Invest in / Captures

#### monthly-debrief ([HH:MM])
- Recap: [confirmed roll-up of the month's weekly recaps, 4-6 lines]
- Momentum / Measures reviewed / Against the goal / Carry forward / Captures

[One `###` header per date, appended chronologically (newest at the bottom). Multiple rhythm entries on the same day nest under one header. Written by `/daily`, `/weekly`, and `/monthly` only; full field rules in their command references. `Recap` lines are user-confirmed and are the base layer for the next roll-up (see `references/rhythm-prompts.md`). A skipped prompt is recorded as `—`; an all-skipped entry is written as `(empty)` so pattern checks can count it. Rhythms are a journal, not a source of record — anything that matters is captured as `A###` / `C###` / Feedback Stream rows and referenced from the `Captures:` line.]

## Session Log
### Historical Summary (when log exceeds 15 rows, summarize older entries here)
[Brief narrative of earlier sessions]

### Recent Sessions
| Date | Commands Run | Key Outcomes |
|------|--------------|--------------|
[rows — 1 line per session]

## Coaching Notes
[Freeform observations the coach should remember between sessions.]
- [date]: [observation]
```
