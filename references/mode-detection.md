# Mode Detection Priority

When no explicit command is given, detect the user's intent and route. Use first match:

1. Explicit command (`/kickoff`, `/log-artifact`, etc.)
1a. "Start my day" / "Morning check-in" / "What's on today" → `/daily pre`. "End of day" / "Wrapping up for the day" / "How did today go" → `/daily debrief`.
1b. "Plan my week" / "What's on this week" / "Week ahead" → `/weekly pre`. "Wrap up the week" / "Weekly review" / "How did this week go" / "Here's my weekly recap" → `/weekly debrief` (fold the pasted recap into Step 1).
1c. "Plan my month" / "Month ahead" / "What should I focus on this month" → `/monthly pre`. "Wrap up the month" / "Monthly review" / "How did this month go" / "Monthly recap" → `/monthly debrief`.
2. PDF path in the message → `/ingest-manager`, `/ingest-stakeholder`, `/ingest-leveling`, or `/prep-*` based on filename hints (`linkedin`, `profile`, `resume` → manager/stakeholder ingest; `leveling`, `ladder`, `rubric`, `competencies` → leveling; `deck`, `review`, `draft`, `roadmap` → prep). If ambiguous, ask.
3. "Prep for a 1:1" / "I have a 1:1 tomorrow" / "preparing for my 1:1 [with my manager]" → `/prep-1-1`
4. "Prep for a joint 1:1 with [manager] and [stakeholder]" / "1:1 with [stakeholder] and my manager" → `/prep-1-1 --with <stakeholder-id>`
5. "Prep for [stakeholder name]" / "Meeting with [stakeholder]" / "Sync with [person] about [topic]" → `/prep-stakeholder <id>`
6. "Roadmap review with [list of names]" / "Meeting with [name1], [name2], and [name3]" / "Review session with [names]" → `/prep-stakeholder <id1> <id2> ...` (resolve each name; add `--with-manager` if the manager is mentioned by name)
7. "I shipped / launched / finished [thing]", or "I want to log [accomplishment]" → `/log-artifact`
8. "I just had a 1:1" / "My manager said" / "In my 1:1 today" / "How did it go" / "Finished my 1:1" → `/log-1-1` (prep-aware; matches any Scheduled prep with `MGR` in the roster)
9. "Debrief [stakeholder]" / "Stakeholder meeting is done" / "Just got out of the meeting with [name]" → `/log-stakeholder-meeting <id>`
10. "The review with [names] is done" (ambiguous when both MGR and K### were in the room) → ask: `/log-1-1` or `/log-stakeholder-meeting`. Default to `/log-stakeholder-meeting` if the question frame is stakeholder-centric.
11. "A stakeholder said" / "Got feedback from [name/team]" / "A reviewer flagged" / "My manager sent a Slack saying..." (no meeting context) → `/log-feedback` (attribution resolves names to `MGR` or `K###` where possible per `references/commands/log-feedback.md`)
12. "How am I doing" / "Am I on track" / "What's my trajectory" / "Review time coming" → `/review-progress`
13. "What should I focus on" / "Which measure needs work" → `/review-measures`
14. "Company just announced / rolled out new OKRs" / "Team priorities changed" → `/log-company-goals`
15. "New manager" / "Stakeholder changed roles" → `/ingest-manager` or `/ingest-stakeholder` (ask which)
16. "Export to interview prep" / "Send artifacts to interview-coach" → `/export-artifacts`
17. "Who's on my team" / "List my stakeholders" / "What's [name]'s K-number" / "Who have I not talked to lately" → `/people` (with filters as inferred: `--stale` for staleness cues, `--role <Type>` when a Relationship Type is mentioned)
18. "I don't trust [name]" / "[name]'s feedback is vague" / "[name] shifted the goalposts again" / "I missed a commitment to [name]" → `/vibe-check <id>` (resolve name to FK; prompt to log the trust-affecting event per the direction and type cued)
19. "How am I doing with [name]" / "What's the vibe with my manager" / "Trust check on [name]" → `/vibe-check <id>` (deep view)
20. "Vibe check" / "Trust status" / "How are my relationships trending" → `/vibe-check` (roster mode)
21. "I promised [X]" / "I told [name] I'd [X]" / "committed to [name] on [X]" (single sentence) → `/commitment add` with pre-filled values (Owner=me; Owed to resolved from the name mentioned; Commitment pre-filled from the sentence)
22. "Here's what happened: ..." / "Multi-sentence paragraph of commitments / feedback / updates" with no rhythm framing → `/commitment --from-paste` (a recap framed as the week's or month's goes to the rhythm debrief per 1b / 1c)
23. "Did I close out [X]" / "What's the status of my commitment to [name]" → `/commitment --to <id>` or `/commitment <C###>` if the ID is referenced
24. "What do I owe people" / "My commitments" / "My open follow-ups" → `/commitment` (list mode)
25. "I missed the deadline on [X]" / "I need to renegotiate [X]" / "The [commitment] is blocked" → `/commitment <C###> <cancel/renegotiate/block>`
26. "I just shipped [X]" where [X] matches an Open commitment → `/commitment <C###> complete` (with the shipped artifact as completion evidence)
27. "Back up state" / "Moving machines" / "Need to export my state" → `/export-state`
28. Session start with existing `coaching_state.md` → `/status`
29. Session start with no `coaching_state.md` → suggest `/kickoff`
30. Otherwise → suggest the rhythm for the time of day (`/daily pre` / `/daily debrief`, `/weekly` on Mon or Fri, `/monthly` in the first or last week of the month); if there's no `coaching_state.md`, suggest `/kickoff`

## Prep vs. debrief disambiguation

Prep intent cues outrank post-hoc cues when both could match. "Prep for the 1:1 I just finished" is ambiguous — ask: "Prepping for an upcoming meeting, or debriefing one that happened?"

Multi-participant cues: if 2+ names are mentioned, confirm the full roster before resolving IDs. Partial-name matches with multiple hits always ask for disambiguation (see `references/meeting-prep-guide.md`).

## Multi-step intents

When a user's request implies a sequence, state the plan and execute sequentially. Offer the next step, don't mandate it.

| Intent | Sequence |
|--------|----------|
| "I just started a new job" / "Set me up" | `/kickoff` → `/ingest-manager` → `/ingest-leveling` → `/ingest-stakeholder` (as needed) → `/status` |
| "Review season coming" / "Review in [N] weeks" | `/log-company-goals` (if stale) → `/review-measures` → `/review-progress` → `/export-artifacts` (if interview-coach-skill in play) |
| "Prep me for interviews" | `/review-measures` → `/export-artifacts` (then user switches to interview-coach-skill) |
| "Manager changed" | `/ingest-manager` → `/log-1-1` for the first new 1:1 when it happens |
| "New quarter" | `/log-company-goals` → `/review-measures` → `/review-progress` |
| "I have a review meeting with my manager and [stakeholder]" | `/prep-1-1 --with <stakeholder-id>` → (meeting happens) → `/log-1-1` (prep-aware) |
| "I have a quarterly review with [names]" | `/prep-stakeholder <ids>` (optionally `--with-manager`) → (meeting happens) → `/log-stakeholder-meeting <any-id-in-roster>` or `/log-1-1` if manager was in room |

**Behavior**: State the plan briefly ("I'll run kickoff, then ingest your manager, then ingest the leveling doc"), execute the first step, offer transitions ("Done with ingest-manager. Ready to move to leveling?"). Respect skip/redirect. If a planned next step diverges from the state-aware recommendation from the just-finished command, follow the plan but note the alternative.

**Session Start co-firing**: If the user opens with a multi-step intent, compress the Session Start greeting and launch the sequence directly.
