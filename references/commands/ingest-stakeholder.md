# ingest-stakeholder — Parse LinkedIn PDF and seed Stakeholder Background

Read `references/pdf-ingestion.md` for the general protocol. Stakeholder-specific wrapper.

Multiple stakeholders are expected. Each gets a `K###` ID and a Stakeholder Details block.

## Flow

1. User provides path + role: `/ingest-stakeholder <path> <role>` or ask interactively.
2. "What's this person's role / team?" and "What's your relationship with them: collaborator, customer, blocker, peer, sponsor?"
3. Read the PDF. Extract per the LinkedIn schema in `pdf-ingestion.md`.
4. Show "What I Extracted" with confidence labels.
5. Walk stakeholder-specific gap questions ONE AT A TIME:
   - Relationship type (confirm or correct)
   - What they want from your team this half; which OKR
   - Interaction cadence
   - What they've praised / pushed back on
   - Recent decisions affecting your work
6. Assign the next `K###` ID.
7. Append row to Stakeholder Backgrounds table.
8. Create Stakeholder Details block with career arc, priorities, working-style signals, open gaps, enrichment history, confidence.
9. Set `Last Interaction` to today only if the user confirms they've recently interacted — otherwise leave blank.
10. Append Session Log row: `ingest-stakeholder | K### [name] added`.

## Multiple stakeholders in one session

If the user wants to add multiple at once, loop the flow — don't try to batch. One at a time keeps the gap questions targeted.

## Updating an existing stakeholder

If the user says "update K003" or the extracted name matches an existing row:

1. Confirm: "This looks like K003 ([name]). Update instead of add?"
2. Walk the diff flow from `ingest-manager.md` (replace / add new / diff + ask).
3. Append to the Enrichment history field: `[date] — <what changed>`.

## Output Schema

```markdown
## Stakeholder Added: K### — [Name]
Relationship: [type] | Team: [team] | Role: [title]

### Extracted from LinkedIn (confidence: Medium)
- Career arc: [one-line summary]
- Prior roles: [list]

### Gap Questions Answered (confidence: High)
- Priorities: [what they want from your team this half]
- Interaction cadence: [frequency]
- Praised: [list]
- Pushed back on: [list]
- Open gaps: [list]

**Recommended next**: `/ingest-stakeholder <path> <role>` for the next key stakeholder, or `/log-artifact` to connect existing work to this stakeholder. **Alternatives**: `/ingest-leveling`, `/log-1-1`
```
