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

### Capture
- `/log-artifact` — Log an accomplishment in XYZ format (quantified Y required)
- `/log-1-1` — Capture 1:1 notes, feedback, follow-ups
- `/log-feedback` — Log standalone feedback from stakeholders / peers / review docs

### Review
- `/review-measures` — Walk each measure, surface evidence coverage, update maturity calls
- `/review-progress` — Score Momentum and Trajectory toward career goal; write Progress Snapshot
- `/log-company-goals` — Refresh Company Context, recompute Priority Weights

### Portability & handoff
- `/export-artifacts` — Export XYZ library for interview-coach-skill
- `/export-state` — Bundle state for backup or machine move
- `/import-state <path>` — Restore from a bundle

### Session
- `/status` — Prescriptive session-start summary
- `/help` — This list

See README.md for the setup walkthrough and portability instructions.
```
