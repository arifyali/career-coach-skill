# commitment — Track accepted commitments through completion or renegotiation

Captures and manages accepted commitments: specific things the user promised to deliver (or someone promised to deliver to them). Status lifecycle, scoring rules, Trust Log coupling, and the paragraph-paste extraction heuristic live in `references/commitment-schema.md` — read that reference before this file.

The core rule: **only accepted commitments go in the table**. A suggestion is a suggestion until the user says "yes, I committed to that." Every capture flow enforces this via an explicit acceptance confirmation step.

## Invocation

```
/commitment                           # list mode: Open + Blocked, Owner=me, sorted by Due
/commitment add                       # guided one-at-a-time capture of a single commitment
/commitment --from-paste              # paragraph-paste extract-then-confirm mode
/commitment C###                      # deep view / edit a specific commitment
/commitment C### complete             # fast-path status flip (prompts for completion evidence)
/commitment C### block                # fast-path (prompts for blocker)
/commitment C### renegotiate          # fast-path (prompts for new terms)
/commitment C### cancel               # fast-path (prompts for cancellation reason)
/commitment --mine                    # only rows where Owner = me
/commitment --waiting                 # only rows where Owner ≠ me (what others owe you)
/commitment --to <id-or-name>         # filter by Owed to (FK resolved)
/commitment --overdue                 # Due passed, Status still Open / Blocked / Renegotiated
/commitment --status <state>          # filter by Status
```

Filters compose: `/commitment --mine --overdue`.

---

## List mode (`/commitment` with no args, or filters only)

Default view: Open + Blocked, Owner = me, sorted by Due ascending (`not agreed` and empty-Due rows last).

### Flow

1. Read Commitments section from state.
2. Apply filters.
3. Compute per-row flag (overdue / blocked-stale / missed-but-not-logged / none).
4. Render + prescriptive nudge if any flag fired.

### Flag rules

- **overdue** — Due is a real date, has passed, Status is Open / Blocked / Renegotiated. Highest-priority flag.
- **blocked-stale** — Status is Blocked AND Last Updated > 14 days ago. The block itself has gone stale.
- **missed-but-not-logged** — Due passed within the last 7 days, Status still Open. Freshest-reminder flag.

Multiple flags per row: show the highest-priority (overdue > missed-but-not-logged > blocked-stale).

### Prescriptive nudges

Pick at most one, in priority order:
- Any overdue → "You have [N] commitments past Due with Status still Open. `/commitment --overdue` to review."
- Any blocked-stale → "[N] Blocked commitments haven't been updated in 14+ days. `/commitment <C###>` to resolve."
- Any missed-but-not-logged → "[N] commitments were due in the last week with Status still Open. `/commitment <C###>` to update."
- Otherwise skip.

### Output

```markdown
## Commitments — [filter description] — [date]

| ID | Commitment | Owner | Owed to | Due | Status | Flag |
|---|---|---|---|---|---|---|
| C001 | [one-line] | me | K002 — Priya Chen | 2026-11-14 | Open | — |
| C004 | [one-line] | me | MGR — [name] | 2026-10-10 | Open | overdue (10d) |
| C007 | [one-line] | K003 — [name] | me | 2026-10-25 | Open | — |

[nudge line if any]
```

No `Recommended next` block unless a nudge fired. List mode is a lookup; its output is the data.

---

## Add mode (`/commitment add`)

Guided one-at-a-time per the non-negotiable sequencing rule.

### Step 1 — Commitment

"What's the commitment? One sentence."

### Step 2 — Owner

"Owner — you (default), or someone else?"

If `me`: proceed. If someone else: resolve to a FK per `references/meeting-prep-guide.md` resolution pattern (exact `MGR` / `K###` → substring name → disambiguate → create inline).

### Step 3 — Owed to

"Who's expecting the result? Give a name, `K###`, `manager`, or a label (`team`, `review committee`, `self`)."

Resolution: same FK pattern. Free-text labels accepted. `self` = private-to-user commitment (e.g., "finish learning spark SQL by end of month" — not a social commitment, but useful to track).

### Step 4 — Due

"Due date? ISO date (e.g., `2026-11-14`), or `not agreed`."

Natural language date hints (`Thursday`, `end of week`, `end of Q4`) are resolved to a concrete date and shown back for confirmation. If the user explicitly says `not agreed`, record it as-is.

### Step 5 — Source

"Where did this come from? Options: `standalone` (default), a Meeting Preps `MTG###`, a Meeting Log row date, or a Feedback Stream row date + source."

If invoked via a debrief hook (see Extraction hooks below), Source is pre-filled and this step is skipped.

### Step 6 — Linked measures (optional)

"Does this commitment demonstrate any performance measure? Show the Performance Measures Tracker as a numbered menu; multiple selections ok; `none` is valid."

### Step 7 — Acceptance confirmation

"**This becomes a tracked commitment**, which means `/vibe-check`, meeting prep, and `/status` will all reference it. Confirm you've actually accepted this commitment (not just a suggestion I made)? yes / no / edit"

Only `yes` writes the row. `no` aborts cleanly. `edit` lets the user revise any field before confirming.

### Step 8 — Write

- Assign next `C###` (max existing + 1, zero-padded to 3 digits).
- Append row to Commitments table.
- Append Details block with Description, empty Status history, placeholder fields.
- If Source is a `MTG###`, append `C###` to that Meeting Log row's `Follow-ups` column.
- If Source is a Feedback Stream row, append `C###` to that row's `Action Taken` column.
- Session Log row: `commitment-add | C###, owed to [owed-to display], due [date]`.

### Output

```markdown
## Commitment Logged — C### — [date]
- Commitment: [one-line]
- Owner: [me or FK + name]
- Owed to: [FK + name or label]
- Due: [date or "not agreed"]
- Status: Open
- Source: [source]
- Linked measures: [M###, M###] or [none]

**Recommended next**: `/commitment` to see your Open list, or back to whatever you were doing. **Alternatives**: `/vibe-check <id>` if you want to log the trust signal from accepting it.
```

---

## Deep mode (`/commitment C###`)

Shows the full table row + Details block + Status history. Prompts for update: "status change (`complete` / `block` / `renegotiate` / `cancel`) / add note / exit."

Add note: appends a free-text line to the Status history with `[date] note — [text]`. Does not change Status.

### Output

```markdown
## Commitment C### — [commitment one-line]
- Owner: [me / FK + name]
- Owed to: [FK + name or label]
- Due: [date]
- Status: [state] (since [date of last transition])
- Source: [source]
- Linked measures: [list]
- Last Updated: [date]

### Status history
- [date] (new) → Open — [reason]
- [date] Open → Blocked — [blocker]
- ...

### [status-specific sections]
- Blocker: [...]
- Completion evidence: [...]
- Renegotiation terms: [...]
- Cancellation reason: [...]

**Update?** `complete` / `block` / `renegotiate` / `cancel` / `note` / `exit`
```

---

## Status flip modes

### `/commitment C### complete`

1. "Any completion evidence? Short note or an `A###` link, both optional."
2. Update Status → `Completed`, Last Updated = today. Append Status history row with evidence note.
3. If Completion evidence is `A###`, append `C###` to that Artifact Details block's "Evidence sources" line (per cross-table integrity rules in `commitment-schema.md`).
4. **Trust Log coupling** (per `references/commitment-schema.md`): propose up-event to Owed-to person's Trust Log, ask yes / no / edit, write if confirmed.
5. Session Log row: `commitment-complete | C###, [owed-to], trust-log: [yes/no]`.

### `/commitment C### block`

1. "What's the blocker?" Required one-line reason.
2. "Expected unblock date?" Optional ISO date.
3. Update Status → `Blocked`, Last Updated = today. Append Status history row with blocker reason.
4. No Trust Log coupling on `block` — blocking is a normal state transition, not a trust event.
5. Session Log row: `commitment-block | C###, blocker: [truncated]`.

### `/commitment C### renegotiate`

1. "What are the new terms? (new Due, scope change, or both)"
2. "Did you renegotiate before the original Due date? yes / no" (auto-detect if Due is in state; confirm with user)
3. Update Status → `Renegotiated` momentarily, then back to `Open` with new Due. Append Status history row with new terms.
4. **Trust Log coupling**: propose neutral `→` event if renegotiated-before-Due, propose down-event `↓` if renegotiated-after-Due passed. User can override arrow.
5. Session Log row: `commitment-renegotiate | C###, new-due [date], trust-log: [yes/no/skip]`.

### `/commitment C### cancel`

1. "Cancellation reason? Options: `missed` / `superseded` / `scope-changed` / `other` (free text)."
2. If `other`, prompt for free-text reason.
3. Update Status → `Cancelled`, Last Updated = today. Append Status history row with reason.
4. **Trust Log coupling** (per `references/commitment-schema.md`): only `missed` triggers a down-event proposal. `superseded`, `scope-changed`, and `other` default to no proposal (user can run `/vibe-check` manually if they want to log one).
5. Session Log row: `commitment-cancel | C###, reason [tag], trust-log: [yes/no/skip]`.

---

## Paragraph-paste mode (`/commitment --from-paste`)

For the capture UX of "paste a rough weekly recap, get commitments + feedback + accomplishments extracted."

### Step 1 — Paste prompt

"Paste a paragraph. I'll extract candidate commitments (what you promised), feedback (what others said), and accomplishments (what you shipped). You'll walk each candidate before anything is written."

### Step 2 — User pastes

Multi-line input accepted. No length limit; longer paragraphs may surface more candidates.

### Step 3 — Extraction

Per the extraction heuristic in `references/commitment-schema.md`:

- Scan for commitment signals (strong + weak).
- Scan for feedback signals (quotes, "manager said", "stakeholder flagged").
- Scan for accomplishment signals (shipped verbs + quantified Y).
- Scan for status updates to existing commitments ("I finally sent that proposal" when C004 is Open and matches).

Present the output in the format specified in `commitment-schema.md`.

### Step 4 — Walk candidates one at a time

For each candidate (in order of category: commitments → feedback → accomplishments → status updates):
- Show the proposed row.
- Ask: `accept` / `edit` / `reject` / `skip`.
- `accept` runs the appropriate capture flow (add-mode for commitments including the acceptance confirmation step, Feedback Stream append for feedback, `/log-artifact` capture for accomplishments, status flip flow for status updates).
- `edit` lets the user adjust pre-filled values first.
- `reject` discards the candidate.
- `skip` moves on, candidate not written but surfaced again if the paragraph is pasted twice (no deduplication across invocations).

### Step 5 — Vague mentions follow-up

After walking strong candidates, show the "Vague mentions" list (weak signals that didn't rise to candidate status). User can `promote N` to upgrade any to a candidate (which then runs through Step 4 for that single item).

### Step 6 — Summary + Session Log

Final output:

```markdown
## Captured from paste — [date]
- Commitments: +[N] (C###, C###)
- Feedback: +[M] rows
- Accomplishments: +[K] (A###, A###)
- Status updates: [L] applied

**Recommended next**: `/commitment` to see your Open list, or `/review-measures` if accomplishments linked measures.
```

Session Log row: `commitment-paste | [N] commitments, [M] feedback, [K] artifacts, [L] status updates`.

---

## Extraction hooks (debrief flows)

`/log-1-1`, `/log-stakeholder-meeting`, and `/log-feedback` each gain an extraction prompt at the end of their Follow-ups (or Action Taken for feedback) step:

> "Any of these are formal commitments you accepted? I'll structure them in the Commitments table."

For each listed follow-up: show the follow-up text + proposed Commitment pre-filled with Owner=me, Owed to pre-filled from the meeting participant or feedback source, Source auto-set to the current Meeting Log row ID or Feedback Stream row date, Due defaults to `not agreed` if unmentioned.

User walks each: `accept` / `edit` / `reject`. Accepted candidates run through the acceptance-confirmation step and write C###.

Behavior if the user declines to promote any follow-up: identical to the pre-Commitments version of the debrief flow. The hook is additive, not replacing.

Cross-reference: the Meeting Log row's `Follow-ups` column gets the C### IDs appended for any promoted commitments. The original follow-up text stays in place.

---

## Boundaries

- Commitments are **the user's record**. They never leave `coaching_state.md` (gitignored). `/export-artifacts` strips Commitments content explicitly (`references/interop-interview-coach.md`).
- Trust Log coupling always requires user confirmation per the explicit-capture design (`references/vibe-check-rubric.md`). No silent writes to Trust Log from commitment status changes.
- The command does not nag. If the user has overdue commitments and declines to update them, the skill flags the pattern in `/status` but doesn't force resolution. See the non-moralizing stance in `references/commitment-schema.md`.
