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

## Stakeholder Backgrounds
| ID | Name | Role | Team | Relationship Type | Priorities | Last Interaction | Open Gaps |
|----|------|------|------|-------------------|------------|------------------|-----------|
[rows — Relationship Type: collaborator / customer / blocker / peer / sponsor]

### Stakeholder Details
#### K001 — [Name]
- Career arc (from LinkedIn PDF):
- Explicit priorities:
- Working-style signals:
- Open gaps:
- Enrichment history: [date — what changed]
- Confidence: [High / Medium / Low]

[repeat per stakeholder]

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

## 1:1 Log
| Date | Participant | Topics | Explicit Feedback | Implicit Signals | Follow-ups | Linked Measures |
|------|-------------|--------|-------------------|------------------|------------|-----------------|
[rows — Participant: `manager` or stakeholder ID `K###`. Explicit Feedback is also mirrored into Feedback Stream automatically.]

## Feedback Stream
| Date | Source | Channel | Verbatim/Paraphrase | Sentiment | Linked Measure | Linked Artifact | Action Taken |
|------|--------|---------|---------------------|-----------|----------------|-----------------|--------------|
[rows — Source: manager / stakeholder ID / peer / recruiter / review doc. Channel: 1:1 / Slack / email / review / all-hands. Sentiment: Positive / Mixed / Critical.]

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
