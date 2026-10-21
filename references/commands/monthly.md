# monthly — Monthly brief and debrief

Monthly ritual that zooms out from `/weekly`. Pre-brief sets the month's focus against the career goal and leveling measures; debrief builds a month recap on top of the month's weekly recaps, works with the user to correct and extend it, then runs `/review-measures` so the "what should I focus on" lens gets a fresh read once a month.

Prompts, routing rules, and the recap roll-up protocol live in `references/rhythm-prompts.md` — read that reference before this file. For debrief, also read `references/commands/review-measures.md` since `/monthly debrief` auto-invokes it.

## Invocation

```
/monthly                 # infer pre vs. debrief from day of month
/monthly pre             # explicit start-of-month brief
/monthly debrief         # explicit end-of-month debrief
```

### Day-of-month heuristic (implicit mode)

- Last 5 days of the month → `debrief` (for the current month)
- Days 1-3 → ask: "Debrief last month, or pre-brief this one?" (if last month has no `monthly-debrief`, lead with debrief; both can run back to back, debrief first)
- Days 4-7 → `pre`
- Otherwise → ambiguous; ask "pre or debrief?"

If a `monthly-pre` entry already exists for the current month, prefer `debrief` on implicit calls.

Which month a debrief covers: on days 1-7, if last month has no `monthly-debrief`, `/monthly debrief` (explicit or implicit) closes out last month. Otherwise it covers the current month.

If `/weekly debrief` is also due today, run it first (or offer to) — the monthly recap builds on it.

---

## Pre-brief flow (`/monthly pre`)

### Step 1 — Dashboard

Read-only surface of the month ahead:

- **Last month's recap**: the `Recap` and `Carry forward` lines from the most recent `monthly-debrief`. If none exists: "No monthly recap yet — this is the first."
- **Career goal + review timing**: short-term Career Goal one-liner; days until `Profile.Review cycle timeline` if set. If the review falls inside this month or next, call it out.
- **Measures to watch**: High-weight measures with Evidence Strength Weak, or with no new linked artifact in 60+ days. Max 3, by Priority Weight then staleness.
- **Commitments due this month**: Open rows with Due in the month.
- **Context freshness**: Company Context age if > 90 days; latest Momentum + Trajectory from Progress Snapshots.

### Step 2 — Structured prompts (3)

Per `references/rhythm-prompts.md` — Monthly pre-brief set. One at a time:

1. **Focus**: "What's the one thing this month should move toward your career goal?" (Offer the Measures to watch and last month's `Carry forward` as options.)
2. **Evidence by month-end**: "What evidence do you want to have by the end of the month — a shipped artifact, a piece of feedback, a conversation?" (Encourage something that would become an `A###` with a quantified Y or a Feedback Stream row on the focus measure.)
3. **Relationship to invest in**: "Whose read on your work matters most this month, and what do they need to see?" (Pre-surface: signal sources on the focus measure, stakeholders flagged by `/people --stale`, Low-trust rows on High-weight measures.)

Apply response-detection rules per `rhythm-prompts.md`. Commitment signals on #2/#3 → inline `/commitment add` candidate. Named meetings on #3 → suggest `/prep-1-1` or `/prep-stakeholder <id>`.

### Step 3 — Write Rhythms entry + Session Log

```markdown
#### monthly-pre ([HH:MM])
- Focus: [response, with M### if resolved]
- Evidence by month-end: [response]
- Invest in: [response, with MGR / K### if resolved]
- Captures: [summary]
```

Session Log row: `monthly-pre | focus [M### or short phrase], [capture summary]`.

### Step 4 — Priority Move for the month

Always emit — the user just named a focus:

> **Priority Move for the month**: [specific action toward the evidence named in #2].

---

## Debrief flow (`/monthly debrief`)

### Step 1 — Build the month recap

Follow the recap roll-up protocol in `references/rhythm-prompts.md`, with the month as the period and weekly recaps as the base layer:

1. **Gather**: every `weekly-debrief` `Recap` line in the month; for weeks without one, that week's daily entries; plus the month's captures (A###, F###, C### status changes, Meeting Log rows, Progress Snapshots) and the `monthly-pre` entry.
2. **Draft**: 4-6 lines — headline outcome, evidence added (by ID), relationships that moved, what stalled, and how the month landed against the `monthly-pre` Focus and Evidence-by-month-end lines.
3. **Show and build**: present the draft, then ask one question at a time: "What's wrong or missing?" → "Anything you wrote yourself this month — a status update, a self-review note — that I should fold in?" Merge what the user gives; anything that reads as a new artifact / feedback / commitment routes to its capture flow per response-detection rules.
4. **Confirm**: show the final recap and get a `yes` / edit before writing. The confirmed text is what future reads build on.

Name gaps plainly: "Weeks of Oct 6 and Oct 13 have no rhythm entries — anything from those weeks I should know?"

### Step 2 — Auto-invoke `/review-measures`

Run `/review-measures` inline per `references/commands/review-measures.md`, scoped by default to: the `monthly-pre` Focus measure, every measure with a new linked artifact or feedback this month, and High-weight measures flagged Stale or Needs evidence. Offer the full walk: "Want the full measure walk instead?"

Don't re-run `/review-progress` — summarize the month's Progress Snapshots instead (Momentum first → last, Trajectory changes). If there are none this month, note it and suggest `/review-progress` as Next.

If `/review-measures` can't run (no tracker), say so and continue.

### Step 3 — Structured prompts (2)

Per `references/rhythm-prompts.md` — Monthly debrief set. These build on the confirmed recap rather than re-asking what it covers:

1. **Against the goal**: "Looking at that recap, did this month move you toward [short-term Career Goal]? What's the honest read?" (Ground in the measure review just shown; disagree directly if the evidence says otherwise, per `references/coaching-voice.md`.)
2. **Change for next month**: "What do you want to do differently next month — keep, stop, or start?"

### Step 4 — Write Rhythms entry + Session Log

```markdown
#### monthly-debrief ([HH:MM])
- Recap: [confirmed recap, 4-6 lines]
- Momentum: [first] → [last] this month, Trajectory [X]
- Measures reviewed: [M### → maturity changes, or "no changes"]
- Against the goal: [response]
- Carry forward: [response to #2 + anything unresolved from the recap]
- Captures: [summary]
```

Session Log row: `monthly-debrief | [N] measures reviewed, [maturity changes], [capture summary]`.

Rhythms archival runs here too — see `references/archival-rules.md`. This month's confirmed recap is the summary source for compressing its daily entries later.

### Step 5 — Priority Move for next month

> **Priority Move for next month**: [specific action grounded in the recap, measure review, and Carry forward].

---

## Output schema

### Pre-brief

```markdown
## Monthly Brief — [Month YYYY]

### Dashboard
- Last month: [recap one-liner or "first month"]
- Career goal: [one line] — review in [N] days
- Measures to watch: [M### list]
- Commitments due this month: [N]

### Prompts
- Focus: [response]
- Evidence by month-end: [response]
- Invest in: [response]

### Captures
[list or "none"]

**Priority Move for the month**: [action].
```

### Debrief

```markdown
## Monthly Debrief — [Month YYYY]

### Recap
[confirmed recap]

[/review-measures output, scoped]

### Momentum this month
[first] → [last], Trajectory [X] ([N] snapshots)

### Reflection
- Against the goal: [response]
- Carry forward: [response]

### Captures
[list or "none"]

**Priority Move for next month**: [action].
```

## Boundaries

- The recap draft is a starting point, never written without the user's confirmation. Don't invent outcomes to fill quiet weeks — name the gap.
- Prompts fire one at a time, including inside the auto-invoked `/review-measures`.
- Monthly debrief does not run commitment flips or the trust review — `/weekly debrief` owns those. If neither has run in 14+ days, suggest `/weekly debrief` as an alternative.
- Rhythms entries never export via `/export-artifacts`.
