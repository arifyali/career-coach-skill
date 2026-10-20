# help — Command list

Show the user the available commands with one-line descriptions. Keep it compact.

## Output

```markdown
## career-coach-skill — Commands

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
