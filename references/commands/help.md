# help — Command list

Show the user the available commands with one-line descriptions. Keep it compact.

`/help` shows the short view (rhythms first, plus the handful of commands people reach for directly). `/help --all` shows the full list.

## Output — `/help`

```markdown
## career-coach-skill — Start here

Most days you only need two commands. They show what's on deck, ask a few questions, and route your answers into artifacts, feedback, and commitments for you.

- `/daily` — Morning pre-brief or end-of-day debrief (inferred from time of day; force with `pre` / `debrief`)
- `/weekly` — Start-of-week pre-brief or end-of-week debrief (recap from your dailies + Momentum and Trajectory read)

### When something specific comes up
- `/prep-1-1` / `/prep-stakeholder <id>` — Prep for a meeting
- `/log-1-1` / `/log-stakeholder-meeting <id>` — Debrief a meeting
- `/status` — Where things stand and the highest-leverage next move

Or just describe what happened in plain language — I'll route it.

`/help --all` for the full command list.
```

## Output — `/help --all`

```markdown
## career-coach-skill — Commands

### Rhythms (start here)
- `/daily [pre|debrief]` — Daily pre-brief or debrief; guided prompts that route into captures
- `/weekly [pre|debrief]` — Weekly pre-brief or debrief; debrief builds a recap on the week's dailies and wraps `/review-progress`

### Setup
- `/kickoff` — Initialize profile, career goals, company context
- `/ingest-manager <pdf>` — Parse manager LinkedIn PDF
- `/ingest-stakeholder <pdf> <role>` — Parse stakeholder LinkedIn PDF
- `/ingest-leveling <pdf-or-paste>` — Extract performance measures from leveling doc

### Meetings — prep
- `/prep-1-1 [--with <ids>] [artifact-path]` — Prep for a 1:1 with your manager (optionally joint). Artifact optional.
- `/prep-stakeholder <id1> [id2 ...] [--with-manager] [--with-skip] [artifact-path]` — Prep for a stakeholder meeting (1 or more stakeholders).

### Capture
- `/log-artifact` — Log an accomplishment in XYZ format (quantified Y required)
- `/log-1-1` — Capture a 1:1 (prep-aware: debriefs against a matching Meeting Prep if one exists)
- `/log-stakeholder-meeting <id-or-name>` — Debrief a stakeholder meeting (prep-aware)
- `/log-feedback` — Log standalone feedback from stakeholders / peers / review docs
- `/commitment [add|<C###>|--from-paste|--mine|--waiting|--to|--overdue|--status]` — Track accepted commitments (Open → Blocked → Completed / Renegotiated / Cancelled); auto-proposes Trust Log entries on status flips

### Review
- `/review-measures` — Walk each measure, surface evidence coverage, update maturity calls
- `/review-progress` — Score Momentum and Trajectory toward career goal; write Progress Snapshot
- `/log-company-goals` — Refresh Company Context, recompute Priority Weights

### Portability & handoff
- `/export-artifacts` — Export XYZ library for interview-coach-skill
- `/export-state` — Bundle state for backup or machine move
- `/import-state <path>` — Restore from a bundle

### Relationships
- `/people [name-substring] [--stale] [--role <Type>] [--trust-flags]` — Look up manager + stakeholder IDs (read-only; includes Trust columns)
- `/vibe-check [id-or-name]` — Trust tracking: roster scan or deep view + log flow for one person

### Session
- `/status` — Prescriptive session-start summary
- `/help` — This list

See README.md for the setup walkthrough and portability instructions.
```
