# Mode Detection Priority

When no explicit command is given, detect the user's intent and route. Use first match:

1. Explicit command (`/kickoff`, `/log-artifact`, etc.)
2. PDF path in the message → `/ingest-manager`, `/ingest-stakeholder`, or `/ingest-leveling` based on filename hints (`linkedin`, `profile`, `resume` → manager/stakeholder; `leveling`, `ladder`, `rubric`, `competencies` → leveling). If ambiguous, ask.
3. "I shipped / launched / finished [thing]", or "I want to log [accomplishment]" → `/log-artifact`
4. "I just had a 1:1" / "My manager said" / "In my 1:1 today" → `/log-1-1`
5. "A stakeholder said" / "Got feedback from [name/team]" / "A reviewer flagged" → `/log-feedback`
6. "How am I doing" / "Am I on track" / "What's my trajectory" / "Review time coming" → `/review-progress`
7. "What should I focus on" / "Which measure needs work" → `/review-measures`
8. "Company just announced / rolled out new OKRs" / "Team priorities changed" → `/log-company-goals`
9. "New manager" / "Stakeholder changed roles" → `/ingest-manager` or `/ingest-stakeholder` (ask which)
10. "Export to interview prep" / "Send artifacts to interview-coach" → `/export-artifacts`
11. "Back up state" / "Moving machines" / "Need to export my state" → `/export-state`
12. Session start with existing `coaching_state.md` → `/status`
13. Session start with no `coaching_state.md` → suggest `/kickoff`
14. Otherwise → ask whether to run `/status`, `/kickoff`, or `/help`

## Multi-step intents

When a user's request implies a sequence, state the plan and execute sequentially. Offer the next step, don't mandate it.

| Intent | Sequence |
|--------|----------|
| "I just started a new job" / "Set me up" | `/kickoff` → `/ingest-manager` → `/ingest-leveling` → `/ingest-stakeholder` (as needed) → `/status` |
| "Review season coming" / "Review in [N] weeks" | `/log-company-goals` (if stale) → `/review-measures` → `/review-progress` → `/export-artifacts` (if interview-coach-skill in play) |
| "Prep me for interviews" | `/review-measures` → `/export-artifacts` (then user switches to interview-coach-skill) |
| "Manager changed" | `/ingest-manager` → `/log-1-1` for the first new 1:1 when it happens |
| "New quarter" | `/log-company-goals` → `/review-measures` → `/review-progress` |

**Behavior**: State the plan briefly ("I'll run kickoff, then ingest your manager, then ingest the leveling doc"), execute the first step, offer transitions ("Done with ingest-manager. Ready to move to leveling?"). Respect skip/redirect. If a planned next step diverges from the state-aware recommendation from the just-finished command, follow the plan but note the alternative.

**Session Start co-firing**: If the user opens with a multi-step intent, compress the Session Start greeting and launch the sequence directly.
