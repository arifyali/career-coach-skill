# daily — Daily brief and debrief

Short ritual to protect against forgetting things. Pre-brief in the morning: dashboard + 2-3 structured prompts + optional paragraph-paste. Debrief at end of day: 2-3 reflective prompts + commitment status flip pass + optional paragraph-paste.

Both modes write a Rhythms entry so `/status` can detect "when was the last daily?" and surface cadence nudges. Prompts and routing rules live in `references/rhythm-prompts.md` — read that reference before this file.

## Invocation

```
/daily                   # infer pre vs. debrief from time of day
/daily pre               # explicit morning brief
/daily debrief           # explicit end-of-day debrief
```

### Time-of-day heuristic (implicit mode)

- Before 12:00 local → `pre`
- 12:00 to 17:00 local → ambiguous; ask: "pre or debrief?"
- After 17:00 local → `debrief`

If the user runs `/daily` and a `daily-pre` entry already exists for today, prefer `debrief` even in the morning (user did pre, is now debriefing — or starting fresh with explicit `/daily pre` to re-brief).

---

## Pre-brief flow (`/daily pre`)

### Step 1 — Dashboard

Read-only surface of today's state. Keep it short — if there's nothing, say so.

- **Meetings today**: Meeting Preps rows with `Date == today` AND `Status: Scheduled`. For each: `MTG### — [names] at [time if captured]`.
- **Commitments due today or in next 48h**: Commitments rows with `Status: Open` AND `Due ≤ today + 2 days`. For each: `C### — [commitment one-line] — Due: [date]`.
- **Stale items**:
  - Overdue commitments (per `references/commands/commitment.md` list-mode flag rules).
  - Meeting Preps overdue for debrief (`Status: Scheduled` AND Date more than 2 days ago).
  - Feedback floor active (per `references/rubric-performance.md`).
  - Decay-pending trust rows (per `references/vibe-check-rubric.md`).

If all four categories are empty: emit "Clear slate — no meetings, no commitments due in the next 48h, no stale items." Skip to Step 2.

### Step 2 — Structured prompts (3)

Per `references/rhythm-prompts.md` — Daily pre-brief set. One at a time:

1. **Intention**: "What's the one thing you want to be true at end of day?"
2. **Biggest risk**: "What's the biggest risk today — a decision, a dependency, a conversation you've been putting off?"
3. **Raise with**: "Who do you need to raise what with today?" (Skill pre-surfaces candidates: recent Trust Log drift sources, stakeholders Owed-to on overdue commitments, Feedback Stream Sources with pending items not yet linked to a Meeting Log row.)

For each response, apply response-detection rules from `rhythm-prompts.md`:
- Shippable outcome → inline `/log-artifact` candidate flow (rare pre-brief; more common in debrief)
- Commitment signal → inline `/commitment add` candidate flow (common on the "Raise with" prompt)
- Feedback signal → inline `/log-feedback` candidate (uncommon pre-brief)
- Trust signal → surface `/vibe-check <id>` suggestion (don't run inline)

### Step 3 — Optional paragraph-paste

"Anything else to capture before you start? (Paragraph is fine; I'll extract commitments / feedback / artifacts.)"

User can `skip`. If they paste, run the extractor per `references/commitment-schema.md`'s paragraph-paste heuristic. Each candidate walks through its destination command's acceptance step.

### Step 4 — Write Rhythms entry + Session Log

Append to `## Rhythms` section:

```markdown
### [today's date]
#### daily-pre ([HH:MM])
- Intention: [response, truncated to one line]
- Biggest risk: [response]
- Raise with: [response]
- Captures: [summary like "1 commitment (C004), 1 feedback (F023)"]
```

If the day's `###` header already exists (e.g., user ran a weekly entry earlier that same day), append the `#### daily-pre` under it. Otherwise create the day header.

Session Log row: `daily-pre | [capture summary or "no captures"]`.

### Step 5 — Priority Move (optional)

If the prompts surfaced a clear single most-leveraged action, emit it at the end:

> **Priority Move**: [specific command + reason].

If no clear Priority Move emerged, skip. Rhythm commands are rituals; forcing a Priority Move on every run degrades signal.

---

## Debrief flow (`/daily debrief`)

### Step 1 — Structured prompts (3)

Per `references/rhythm-prompts.md` — Daily debrief set. One at a time:

1. **Shipped**: "What shipped today?"
2. **Unresolved**: "What's unresolved — rolling into tomorrow?"
3. **Surprises**: "Anyone or anything surprise you today?"

Apply response-detection rules per `rhythm-prompts.md`:
- Shippable outcome in response to #1 → inline `/log-artifact` candidate flow with quantified-Y guardrail
- Commitment signal (something unresolved that was actually a commitment) → inline `/commitment add`
- Feedback signal → inline `/log-feedback`
- Trust signal in #3 (if a named stakeholder's behavior surprised the user) → surface `/vibe-check <id>` suggestion

### Step 2 — Commitment status flip pass

Read Commitments rows where `Status: Open` AND (`Due == today` OR `Last Updated` within last 7 days — i.e., actively worked on this week).

For each row:
> "C### — [commitment one-line] — Due: [date] — Status: Open. Flip to: `complete` / `block` / `renegotiate` / `unchanged` / `skip`?"

Each non-unchanged flip runs through the status-flip sub-flow in `references/commands/commitment.md` (completion evidence prompt, blocker prompt, renegotiation terms, cancellation reason), including the Trust Log coupling proposal with yes/no/edit confirmation.

If no commitments match, skip Step 2 silently.

### Step 3 — Optional paragraph-paste

"Dump a paragraph for anything else today — meetings held, feedback received, decisions made."

User can `skip`. Full extractor if they paste.

### Step 4 — Write Rhythms entry + Session Log

Append to `## Rhythms` section:

```markdown
#### daily-debrief ([HH:MM])
- Shipped: [response, one line]
- Unresolved: [response]
- Surprises: [response]
- Captures: [summary]
```

Place under today's `###` date header (creating if needed; appending if the day already has entries).

Session Log row: `daily-debrief | [capture summary], [N] commitment flips`.

### Step 5 — Priority Move (optional)

If an unresolved item from Step 1 or Step 3 clearly needs tomorrow's attention, emit:

> **Priority Move for tomorrow**: [specific action].

Skip if nothing stands out.

---

## Output schema

Keep it short — rhythms are rituals, not deep workflows.

### Pre-brief

```markdown
## Daily Brief — [date]

### Dashboard
- Meetings today: [N]
- Commitments due in next 48h: [N]
- Stale items: [brief list or "none"]

### Prompts
- Intention: [response]
- Biggest risk: [response]
- Raise with: [response]

### Captures
[list of A###, C###, F### written during the run, or "none"]

[optional **Priority Move**: ...]
```

### Debrief

```markdown
## Daily Debrief — [date]

### Prompts
- Shipped: [response]
- Unresolved: [response]
- Surprises: [response]

### Commitment status
- C###: [prior] → [new]
- C###: unchanged

### Captures
[list of A###, C###, F### written, or "none"]

[optional **Priority Move for tomorrow**: ...]
```

## Boundaries

- Prompts fire one at a time. Never show the user all three up front.
- The ritual is user-driven. If the user types `skip` on every prompt, the Rhythms entry is written as "empty." The skill does not nag.
- Pattern of 3+ empty daily entries in a week → surface once at `/weekly debrief` per the non-moralizing stance in `references/rhythm-prompts.md`.
- Rhythms entries never export via `/export-artifacts` per `references/interop-interview-coach.md`.
