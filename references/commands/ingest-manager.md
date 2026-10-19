# ingest-manager — Parse LinkedIn PDF and seed Manager Background

Read `references/pdf-ingestion.md` for the general protocol. This file is the manager-specific wrapper.

## Flow

1. User provides absolute path to LinkedIn PDF. "What's the path to your manager's LinkedIn PDF? (Save their profile as PDF from LinkedIn's '…' menu.)"
2. Confirm the path exists and the file is a `.pdf`. If not, offer the paste-text fallback.
3. Read the PDF with the Read tool.
4. Extract per the LinkedIn schema in `pdf-ingestion.md`:
   - Name, Headline, Current title+company+start date
   - Experience timeline (title, company, dates, scope)
   - Education, Skills, Certifications, Publications
   - About / Bio paragraph
5. Show a "What I Extracted" summary with per-field confidence labels (High for anything in the PDF text, Medium for anything inferred).
6. Walk gap questions ONE AT A TIME, in the priority order from `pdf-ingestion.md`:
   - Motivation signals (why they moved roles)
   - What they talk about as their wins
   - Invisible scorecard — what they measure that's not in the leveling doc
   - Working-style signals
   - Upstream pressure
   - Explicit priorities for the next quarter
   - Known friction / open gaps
7. Stop when the user says "that's enough" or when all applicable questions are answered.
8. Write to `Manager Background` section:
   - PDF-sourced fields -> `Career arc`, `Prior companies / roles`. Confidence Medium.
   - User-confirmed fields -> `Explicit priorities`, `Working-style signals`, `Open gaps`. Confidence High for the fields the user answered specifically.
   - `Last enriched` = today.
9. Append Session Log row: `ingest-manager | parsed [name]'s LinkedIn, enriched [N] fields`.

## Re-ingestion

If `Manager Background` already has data (user re-ran the command):

1. "You already have manager data on file. Options: (a) replace entirely, (b) add new fields only, (c) diff first and I'll ask about each change."
2. Default: (c). Show a side-by-side of existing vs. extracted values.
3. For each field with a difference, ask whether to update, keep existing, or merge.
4. Preserve prior values in Coaching Notes if a major change: "[date]: Manager Background updated — prior career arc / priorities archived."

## Output Schema

```markdown
## Manager Ingested: [Name]

### Extracted from LinkedIn (confidence: Medium)
- Current: [title] at [company] since [date]
- Career arc: [one-line summary]
- Notable transitions: [list]

### Gap Questions Answered (confidence: High)
- Explicit priorities for next quarter: [list]
- Working-style signals: [list]
- Open gaps / known friction: [list]

### Still Unknown
- [any fields neither extracted nor answered]

**Recommended next**: `/ingest-leveling <path>` — now that we know their priorities, let's ingest the leveling doc. **Alternatives**: `/log-1-1`, `/ingest-stakeholder`
```
