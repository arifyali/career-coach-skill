# log-company-goals — Refresh company/team priorities

Update Company Context mid-cycle. Fires at quarterly OKR rollouts, reorgs, strategy shifts, or whenever the user says "priorities just changed." Recomputes Priority Weight on every row in Performance Measures Tracker.

## Flow

### Step 1 — What changed

"What changed — company priorities, team priorities, your role's mapping to them, or all of the above?"

### Step 2 — Capture the new context

Depending on the answer, re-run the relevant subset of the `/kickoff` Company Context questions:

- Company mission (changes rarely — only ask if the user signals it did)
- Team mission (changes on reorgs)
- Current-cycle **company** priorities / OKRs
- Current-cycle **team** priorities / OKRs
- Source + Last refreshed = today

### Step 3 — Role-to-priority mapping

Always re-ask: "Which 1-3 of the new priorities most depend on your role?" Even if the user only changed team priorities, the mapping often shifts.

### Step 4 — Preview Priority Weight changes

Before writing, show a diff:

```
M001 — [name]: Priority Weight High -> High (no change)
M003 — [name]: Priority Weight Med -> High (now serves team OKR "ship risk-model refresh")
M005 — [name]: Priority Weight High -> Med (no longer in current cycle)
M007 — [name]: Priority Weight Low -> Low (no change)
```

Ask: "These Priority Weight changes will affect Momentum math. OK to apply?"

### Step 5 — Write

- Update Company Context fields.
- Update Priority Weight on every affected Performance Measure row.
- Append Session Log row: `log-company-goals | refreshed Company Context, Priority Weight changes on [N] measures`.

### Step 6 — Coaching commentary

Response blueprint: What I Heard → What Is Working → Gaps To Close → Priority Move → Next Step.

Call out:
- **Realignment opportunity**: measures that newly became High-weight and have Weak evidence are the biggest leverage points for the next review cycle. Name them.
- **De-prioritization**: measures that dropped from High to Med/Low free up attention. Don't ignore them (they're still in the leveling doc), but reset expectations.
- **Review-window implications**: if the user's review window is approaching and High-weight set just changed, flag that the user needs to rebalance evidence quickly.

## Output Schema

```markdown
## Company Context Refreshed — [date]

### What Changed
- [bulleted summary]

### Role-to-Priority Mapping
- [the new 1-3 priorities your role is on the hook for]

### Priority Weight Impact
- [diff from Step 4]

### Realignment
- **Newly High-weight, weak evidence**: [M###, M###] — these are your highest-leverage next artifacts
- **De-prioritized**: [M###, M###] — still in the ladder, but not this cycle's focus

**Recommended next**: `/log-artifact` targeting [specific M###] — [reason]. **Alternatives**: `/review-measures`, `/review-progress`
```
