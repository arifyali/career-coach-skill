# kickoff — Setup Workflow

Initialize a user's coaching profile, career goals, and company context. Also runs the Mid-Role Update flow when re-invoked after a role change.

## First-run bootstrap

Before running the question flow:

1. Check whether `coaching_state.md` exists at the skill root.
2. If NOT: copy `coaching_state.example.md` to `coaching_state.md`. This is the empty template with the schema skeleton — never commit this copy (it's gitignored).
3. If it DOES exist and the user re-ran `/kickoff`: branch to the **Mid-Role Update** flow below.

## Pre-start detection

Watch for signals that the user hasn't started the job yet. Any of these flip the flow to the **Pre-start variant** at the bottom:

- The user volunteers it directly ("I haven't started yet", "my start date is X").
- Step 1 Q5 (Review cycle timeline) comes back as unknown AND Steps 1 Q1-Q4 named a company the user is joining, not working at.
- Company Context fields come back as TBD because the user hasn't been onboarded.

The pre-start variant doesn't replace the main flow — it adjusts seeds, defaults, and the first-plan shape in Steps 2, 4, 5, and 7. Full details in the **Pre-start variant** section below.

## Step 1 — Role + level

One question at a time:

1. "Which role profile fits you? Options: `data-scientist`, `analytics-engineer`, `bi-engineer`, `data-pm`, `generic-data`." Show the list from `references/role-profiles.md` and let the user pick. Write the key to `Profile.Role profile source`.
2. "What's your current level?" (e.g., L3, Senior, IC2 — accept whatever the company uses)
3. "What's your target next level?" (next promotion rung, or a lateral move target like "transition to Data PM")
4. "What company?"
5. "When is your next formal review cycle?" (date or "ongoing")
6. "When did you start this role — or when do you start?" Write to `Profile.Role start date`. If tenure is under 180 days (or the date is in the future), say: "You're inside the ramp window, so for the first 90 days progress reads track ramp checkpoints instead of the Momentum thresholds. Want a different ramp length (60-180 days)?" Write the answer to `Profile.Ramp period (days)` (default 90). If tenure is already past the ramp length, write 90 without asking — the ramp won't apply. See Ramp Period in `references/rubric-performance.md`.

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
- Role start date: [date] — ramp: [day N of 90 / starts [date] / complete]
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

## Pre-start variant

Triggered by the detection rules above. Apply these adjustments to the main flow:

### Step 1 Q5 adjustment — Review cycle timeline

The user doesn't know yet. Record `TBD — establish in first 2-3 weeks on the job` and add a Coaching Note:
> [date]: Review cycle timeline TBD. Establish in first 2-3 weeks on the job via first manager 1:1; update `Profile.Review cycle timeline` + `Career Goals.Short-term.Target review window`.

### Step 2 adjustment — Short-term career goal seed

The default `"Promote to [target level] by [review window]"` seed is misleading pre-start — the user has no evidence base yet, and a 6-12 month promotion horizon from a start date that hasn't happened is almost always too optimistic. Future `/review-progress` calls would read "Lagging" for the entire first 6 months even if ramping on track.

Use this seed instead:

> **Land cleanly in [current level] at [company] and build the evidence base for the [target level] promotion case.**

With pre-start-shaped success criteria:
- First 30 days: team / stakeholder map complete; measures ingested; first artifact logged
- First 90 days: shipped at least one end-to-end project with quantified Y
- 6 months: 3+ artifacts against each High-Priority-Weight measure; manager-confirmed on track for [target level] case

Offer the seed; let the user edit or replace. The important thing is that the goal reflects "land + build case" shape, not "promote by date" shape.

### Step 4 adjustment — Company Context batching

After the first TBD answer in Step 4, offer to batch-TBD the rest:

> I'll mark the remaining Company Context fields (team OKRs, source, confidence) all `TBD` and jump to Step 5 — or we keep going one question at a time if you have more signal on team OKRs than company-wide. Which?

If the user accepts the batch, mark Company Mission / Team Mission / Company OKRs / Team OKRs / Source all `TBD` and add a Coaching Note:
> [date]: Pre-start user. Start date [date]. All Company Context fields TBD — fill via `/log-company-goals` in first 2-3 weeks after start, **before** `/ingest-leveling`, so Priority Weights land correctly on measures from day one.

### Step 5 adjustment — Role-to-priority mapping

If Company Context is uniformly TBD, Role-to-priority mapping has nothing to anchor against. Record `Role-to-priority mapping: TBD` and extend the Step 4 Coaching Note:
> Role-to-priority mapping TBD. Re-ask once `/log-company-goals` fills Company Context; recompute Priority Weights across all measures.

### Step 7 adjustment — First-session recommendations

Replace the standard First Plan with the pre-start plan. The pre-start Priority Move is `/ingest-manager`, not `/ingest-leveling` — the leveling doc isn't accessible yet, but the public manager LinkedIn is.

```markdown
## First Plan

### Now (pre-start, before [start date])
1. `/ingest-manager <path-to-manager-linkedin.pdf>` — Priority Move; sets up every downstream prep + debrief flow and sharpens the first 1:1
2. `/prep-1-1` (optional dry run) — surfaces what the skill *would* ask with empty Manager Background; a second run after `/ingest-manager` is meaningfully sharper

### First week on the job ([start date] → ~[start date + 7])
3. `/log-company-goals` — fill all the TBD Company Context fields; **run this before step 4** so Priority Weights compute correctly
4. `/ingest-leveling <leveling-doc.pdf>` — generates the Performance Measures Tracker
5. `/log-1-1` to debrief the first manager 1:1; extract the review cycle timeline and update Profile

### First 30 days
6. `/ingest-stakeholder <linkedin.pdf>` for each key stakeholder met
7. First `/log-artifact` once something ships (quantified Y rule applies from day one)

### Before the promotion case
8. Monthly `/review-progress` to track Momentum and Trajectory against the "land cleanly + build case" goal

**Recommended next**: `/ingest-manager <path-to-manager-linkedin.pdf>` — public LinkedIn is enough. Builds priors for the first 1:1 before day one. **Alternatives**: `/prep-1-1` (dry run), `/help`
```

### Pre-start Coaching Notes summary

By end of kickoff, a pre-start user should have at least these Coaching Notes:
- Pre-start user + start date + Company Context batched to TBD
- Review cycle timeline TBD; establish in first 2-3 weeks on the job
- Role-to-priority mapping TBD; re-ask after `/log-company-goals`
- Any long-term career goal uncertainty worth revisiting at a specific milestone (e.g., SDS II case)

Session Log row still fires normally: `kickoff | Profile + Career Goals + Company Context initialized (pre-start user; most Company Context TBD)`.

## Mid-Role Update

Users re-run `/kickoff` after a role change, promotion, or team move. Don't restart from scratch.

1. Ask: "What's changed? Role, level, company, team, manager, or something else?"
2. Show what carries over: "Your Artifact Library and Meeting Log stay. Performance Measures Tracker: ask whether to deprecate current rows (if the role or leveling doc changed) or keep them (if just a team move)."
3. Update `Profile` fields. Preserve old values as a Coaching Note: "[date]: Previous role: [X] at [company]; current level [Y]; biggest concern [Z]."
   - New company or new team: ask for the new `Role start date` (restarts the ramp). Promotion in place: ask whether to restart the ramp; default no, since the user keeps their context.
4. Re-run Steps 4-5 (Company Context and Role-to-priority mapping) if the move was to a different team/company.
5. Flag downstream work:
   - New leveling doc needed? Prompt for `/ingest-leveling`.
   - New manager? Prompt for `/ingest-manager`.
   - Priority Weight needs recomputation? Prompt for `/log-company-goals`.

Output a brief "Profile Update Summary" showing what changed, what carries over, and the 2-3 highest-priority actions.
