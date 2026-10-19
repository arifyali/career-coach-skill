# ingest-leveling — Parse leveling doc and seed Performance Measures Tracker

Read `references/pdf-ingestion.md` for the general protocol. Leveling-specific wrapper.

The leveling doc is often the single most important artifact in the skill — it defines what the user is measured against. Getting this right pays off every subsequent command.

## Flow

### Step 1 — Path and format

"What's the path to your leveling doc / ladder / rubric? PDF or pasted text both work."

- If PDF: Read with the Read tool. If >10 pages, ask the user which pages are relevant.
- If pasted: capture directly.
- If no doc exists: fall back to role-profile seed measures from `role-profiles.md` and ask the user to supplement with any informal criteria they know. Mark `Source: role-profile seed` on all rows.

### Step 2 — Confirm context

Before extracting:

- "What's your current level in this doc?"
- "What's your target next level?"

Only extract criteria at the target level. Current-level criteria are presumed (if you can't demonstrate the current level, you wouldn't be at it).

### Step 3 — Extract

Follow the extraction schema in `pdf-ingestion.md`. Walk the doc and produce draft Performance Measure rows:

- `Measure`: concise action-verb restatement of each criterion (1-2 lines). Not a direct copy.
- `Source`: section + heading path in the doc. Precise enough that the user can find the original.
- `Current Maturity`: `Not Yet Demonstrated`.
- `Target Maturity`: `Meeting` (default; `Exceeding` if user explicitly asks).
- `Trend`: `Insufficient Signal`.
- `Priority Weight`: compute from `Company Context.Role-to-priority mapping` per `rubric-performance.md`. If Company Context is empty, default to `Med` and leave a Coaching Note.
- `Supporting Artifact IDs`: empty.
- `Last Reviewed`: empty.

### Step 4 — Preview and refine

Show the extracted measures as a numbered list. Walk them with the user:

- "Any that are too vague to be useful? I'll ask you what your manager says they look like concretely." (Gap question for ambiguous criteria.)
- "Any that are missing — things your manager explicitly measures that aren't in this doc?"
- "Any that are irrelevant to your role within the broader category?"
- "Which 2-3 are the ones your manager has flagged as most important for your next review?" (overrides automatic Priority Weight with user-identified priority = High.)

### Step 5 — Assign IDs and write

- Starting from `M001` on first ingest, or from max existing + 1 on re-ingest.
- Append rows to Performance Measures Tracker.
- Set `Role profile source` in Profile if the doc implies a different role than previously recorded; ask the user to confirm.
- Append Session Log row: `ingest-leveling | parsed [doc name], [N] measures added`.

## Re-ingestion

Companies update leveling docs. On re-ingest:

1. Diff the new doc against existing Performance Measures Tracker.
2. Show "What's new / changed / removed" before writing anything.
3. For removed criteria: append `(deprecated [date])` to the Measure field but preserve the row and its Supporting Artifact IDs. Don't delete.
4. For new criteria: new `M###` rows with `Supporting Artifact IDs` empty.
5. For reworded criteria: ask whether to replace the Measure text or keep both as a historical note.

## No leveling doc available

If the user has no leveling doc:

1. "Does your manager have an informal rubric? A set of expectations in writing anywhere — team doc, PR template, promotion criteria email thread?"
2. If yes: paste it in, run the normal flow.
3. If no: use `role-profiles.md` seed measures. Flag in Coaching Notes: "[date]: No leveling doc available — using role-profile seeds. Request a formal rubric from manager in next 1:1."

## Output Schema

```markdown
## Leveling Ingested: [doc name or "role-profile seeds"]
Current level: [X] | Target: [Y] | Source: [doc title / role-profile seed]

### Measures Added ([N])
| ID | Measure | Priority Weight | Source |
|----|---------|-----------------|--------|
...

### Flagged for Clarification
- [any ambiguous criteria + the gap question that'll fire in a future 1:1]

### Priority Mapping
- High-weight measures (your anchors): [list]
- Med/Low: [count]

**Recommended next**: `/log-artifact` — seed the library with 2-3 recent accomplishments against your High-weight measures. **Alternatives**: `/ingest-stakeholder`, `/log-1-1`, `/review-measures`
```
