# kickoff — Setup Workflow

Initialize a user's coaching profile, career goals, and company context. Also runs the Mid-Role Update flow when re-invoked after a role change.

## First-run bootstrap

Before running the question flow:

1. Check whether `coaching_state.md` exists at the skill root.
2. If NOT: copy `coaching_state.example.md` to `coaching_state.md`. This is the empty template with the schema skeleton — never commit this copy (it's gitignored).
3. If it DOES exist and the user re-ran `/kickoff`: branch to the **Mid-Role Update** flow below.

## Step 1 — Role + level

One question at a time:

1. "Which role profile fits you? Options: `data-scientist`, `analytics-engineer`, `bi-engineer`, `data-pm`, `generic-data`." Show the list from `references/role-profiles.md` and let the user pick. Write the key to `Profile.Role profile source`.
2. "What's your current level?" (e.g., L3, Senior, IC2 — accept whatever the company uses)
3. "What's your target next level?" (next promotion rung, or a lateral move target like "transition to Data PM")
4. "What company?"
5. "When is your next formal review cycle?" (date or "ongoing")

## Step 2 — Short-term career goal

Ask: "What's your short-term career goal — the next 6-12 months?"

Default seed: `"Promote to [target level] by [review window]"` built from Step 1 answers. Offer this to the user: "Does that sound right, or do you want to replace it with a different concrete milestone — scope expansion, specific skill mastery, role switch?"

If the goal is vague ("get better at modeling", "grow"), press for success criteria: "What specifically would be different in 6-12 months if this goal lands? Give me 1-3 observable checkpoints." If the user resists, note it in Coaching Notes and flag that `/review-progress` will produce a weaker Trajectory Call until criteria exist.

Write to `Career Goals.Short-term`:
- Goal
- Success criteria (1-3 bullets)
- Target review window
- Last reviewed = today

## Step 3 — Long-term career goal (optional)

"Any long-term direction — 2-5 years out? This is optional. Say `skip` if you don't have one yet."

If given, write to `Career Goals.Long-term`. If skipped, leave empty. Not every user has a 5-year plan and faking one is noise.

## Step 4 — Company Context

One question at a time:

1. "What's the company mission — in a sentence?"
2. "What's your team's mission?"
3. "What are the current-cycle **company** priorities or OKRs? Doc link, paste, or summary from memory — whichever you have."
4. "What are the current-cycle **team** priorities or OKRs?"
5. "Where did this info come from — all-hands, team doc, manager 1:1?" (`Source` field)
6. Confidence: default `Medium` if from memory, `High` if the user pasted a doc.

Write to `Company Context` section.

## Step 5 — Role-to-priority mapping

"Of the company/team priorities you just listed, which 1-3 most depend on your role?"

Write to `Company Context.Role-to-priority mapping`. This field is critical: it feeds the Priority Weight on every performance measure that gets ingested later.

## Step 6 — Preferences

1. "On a 1-5 scale, how direct should my feedback be? 5 = maximum directness, 1 = heavy encouragement framing. Default is 5." Write to `Profile.Feedback directness`.
2. "What's your biggest career concern right now?" Write to `Profile.Biggest career concern`. If no concern, write `none stated`.
3. "Are you running this for yourself or for a mentee?" Write to `Profile.Mentee flag`.
4. "Where should artifact exports go for interview prep? Default: `../interview-coach-skill/inbox/career-artifacts.md`. If you don't use interview-coach-skill, say `none`." Write to `Profile.Interview-coach export path`.

## Step 7 — First-session recommendations

After kickoff, surface the next three high-value moves:

1. `/ingest-manager <path-to-manager-linkedin.pdf>` — enriches Manager Background
2. `/ingest-leveling <path-to-leveling-doc.pdf>` — generates Performance Measures Tracker
3. `/ingest-stakeholder <path> <role>` — for each key stakeholder

Write a Session Log row: `kickoff | Profile + Career Goals + Company Context initialized`.

## Output Schema

```markdown
## Kickoff Summary
- Role / track: [role] (profile key: `role-profile-source`)
- Current -> Target: [current level] -> [target level]
- Company: [company], review cycle: [date or "ongoing"]
- Feedback directness: [1-5]
- Mentee flag: [self / mentee <handle>]

## Career Goals
- **Short-term**: [goal] — criteria: [criteria]. Window: [date]
- **Long-term**: [goal or "not set"]

## Company Context
- Company mission: [one line]
- Team mission: [one line]
- Current priorities: [1-3 bullets]
- Role maps to: [the 1-3 priorities you'll be measured against]

## First Plan
### Immediate (this session or next)
1. `/ingest-manager <path>` — [reason]
2. `/ingest-leveling <path>` — [reason]

### This week
3. `/ingest-stakeholder <path> <role>` — for each key stakeholder
4. First `/log-artifact` to seed the Artifact Library

### Before your review
5. `/review-progress` to see Momentum and Trajectory

**Recommended next**: `/ingest-leveling <path>` — [reason tied to review window]. **Alternatives**: `/ingest-manager`, `/log-artifact`, `/help`
```

## Mid-Role Update

Users re-run `/kickoff` after a role change, promotion, or team move. Don't restart from scratch.

1. Ask: "What's changed? Role, level, company, team, manager, or something else?"
2. Show what carries over: "Your Artifact Library and Meeting Log stay. Performance Measures Tracker: ask whether to deprecate current rows (if the role or leveling doc changed) or keep them (if just a team move)."
3. Update `Profile` fields. Preserve old values as a Coaching Note: "[date]: Previous role: [X] at [company]; current level [Y]; biggest concern [Z]."
4. Re-run Steps 4-5 (Company Context and Role-to-priority mapping) if the move was to a different team/company.
5. Flag downstream work:
   - New leveling doc needed? Prompt for `/ingest-leveling`.
   - New manager? Prompt for `/ingest-manager`.
   - Priority Weight needs recomputation? Prompt for `/log-company-goals`.

Output a brief "Profile Update Summary" showing what changed, what carries over, and the 2-3 highest-priority actions.
