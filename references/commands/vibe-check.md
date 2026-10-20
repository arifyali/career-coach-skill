# vibe-check — Trust tracking for managers and stakeholders

Capture and review trust signals on a specific person, or scan the roster for trust red flags. Two directions tracked: **Trust (them→you)** — reliability of their signal to the user — and **Trust (you→them)** — the user's follow-through with them. Full rubric lives in `references/vibe-check-rubric.md`; read it before this file at command invocation.

Opt-in by design. `/log-feedback`, `/log-1-1`, and `/log-stakeholder-meeting` do **not** prompt for trust signals — trust capture happens here or not at all.

## Invocation

```
/vibe-check                     # roster mode: table + flags across all people
/vibe-check <id-or-name>        # deep mode: one person, log + view trust log
```

## Roster mode (no args)

### Flow

1. Read Manager Background + Stakeholder Backgrounds from state.
2. Apply decay rule from `references/vibe-check-rubric.md` to each row where Last Interaction (or Last enriched for MGR) > 90 days ago. For each decay event: record a Trust Log entry; move trust toward Medium.
3. Compute per-row flags (see below).
4. Render the table + any prescriptive nudges.

### Flag rules

Compute per person, show in the Flag column:

- **"[N] negative events in 30d"** — 2+ Trust Log entries with ↓ in the last 30 days (either direction).
- **"[N] missed commitments in 60d"** — 2+ you→them ↓ events typed `missed commitment` or `dropped thread` in 60 days.
- **"decay pending"** — Last Interaction > 90 days ago AND trust not already at Medium both directions (user hasn't run `/vibe-check` recently to apply decay — surfaces the stale state).
- **"low-trust signal source on M###"** — this person is the source on 2+ Feedback Stream rows linked to a High-weighted measure M###, AND their them→you trust is Low. Highest-priority flag; career-progression risk.

Multiple flags: show the single most important per row (priority: low-trust signal source > negative events > missed commitments > decay pending).

### Filters

- `/vibe-check --flagged` — only rows with a non-empty Flag column.
- `/vibe-check --low` — only rows with any Low in either direction.
- `/vibe-check --role Reviewer` — reuses canonical Relationship Type enum. Manager row always shown unless combined with `--low` or `--flagged` filters that exclude it.

### Prescriptive nudges

Pick at most one, in priority order:

- Any low-trust signal source on a High-weight measure → "[name] is a signal source on M### (High weight) and at Low trust. Prep extra evidence; `/vibe-check <id>` to review the trust log before `/review-progress`."
- Any missed-commitments flag → "[name] shows [N] missed commitments on your side in 60 days. `/vibe-check <id>` to review the pattern before the next 1:1 or meeting."
- Any decay-pending → "[N] people have gone 90+ days without interaction. Decay applied this run; `/vibe-check <id>` on each to re-establish."
- Otherwise skip the nudge — no news is good news.

### Output

```markdown
## Vibe Check — Roster — [date]

| ID | Name | Trust (them→you) | Trust (you→them) | Last Event | Flag |
|---|---|---|---|---|---|
| MGR | [name] | Med | High | 2026-10-15 ↑ kept commitment | — |
| K001 | [name] | High | Med | 2026-09-28 ↑ specific feedback | — |
| K004 | [name] | Low | Med | 2026-10-18 ↓ goalpost shift | 3 negative events in 30d |
| K007 | [name] | Med | Med | 2026-07-14 ↑ delivered on ask | decay pending (98 days) |

[nudge line if any]
```

No `Recommended next` block unless a nudge fired. Roster mode is a lookup; its output is the data.

## Deep mode (`/vibe-check <id-or-name>`)

### Step 1 — Resolve participant

Use the FK resolution pattern from `references/meeting-prep-guide.md`:

1. Exact `MGR` or `K###` match.
2. Case-insensitive substring match against Stakeholder Backgrounds Name + Manager Background Name.
3. Multiple matches → list candidates (ID, Name, Role, Team, Relationship Type) and ask.
4. No match → "No stakeholder on file matching [input]. `/ingest-stakeholder <pdf>` to add them, or `/people` to see who's on file."

### Step 2 — Decay check

If Last Interaction > 90 days ago, apply decay before showing state:
- Compute old/new scores per the rubric.
- Append Trust Log entry: `[today] decay — no interaction in [N] days; [direction]: [old] → [new]`.
- Write to state.
- Mention in the Step 3 output.

### Step 3 — Show current state

```markdown
## Vibe Check — [Name] ([ID], [Relationship Type for stakeholders])

- Trust (them→you): [Low / Med / High]
- Trust (you→them): [Low / Med / High]
- Last Interaction: [date] ([N] days ago)
- Last 3 Trust Log events:
  - [date] [direction] [↑/↓/→] — [reason]
  - [date] [direction] [↑/↓/→] — [reason]
  - [date] [direction] [↑/↓/→] — [reason]

### Interpretation
[one-line pattern + tactical implication, drawn from the Trust Log pattern]
```

If a decay event just fired, note it: "Decay applied this run: [direction] [old] → [new]."

### Step 4 — Log a trust signal (optional)

"Log a trust signal? (yes / no / show full log)"

- `no` → stop here; emit Priority Move + Next Step.
- `show full log` → render the full Trust Log for this person, then re-ask.
- `yes` → proceed to Step 5.

### Step 5 — One question at a time

Per the non-negotiable sequencing rule:

1. **Direction**: "them → you" or "you → them"?
2. **Event type**: show the direction-specific menu from `references/vibe-check-rubric.md`. User picks a numbered option or types `free text`.
3. **Impact**: default inferred from event type (per the rubric). Show the default + offer override: "Default impact: ↓. Accept, or override to ↑ / →?"
4. **Reason**: "One-line reason — what specifically happened? (becomes the Trust Log row)"

### Step 6 — Write

- Update the Trust column on Stakeholder Backgrounds (or Manager Background field) per the impact + bounding rules (High can't go higher; Low can't go lower; one notch max per event).
- Append Trust Log row to the person's Details block (Stakeholder Details or Manager Trust Log).
- Append Session Log row: `vibe-check | [id] [direction] [impact] — [reason, truncated to ~40 chars]`.

### Step 7 — Reinterpret

Show the updated state + a one-line interpretation. If the event tripped a flag (now at Low, now has 2+ negative events in 30d, etc.), surface it with a tactical next action.

Response blueprint (tight, this is a diagnostic command):

- **What I Heard**: 1 line
- **Priority Move**: 1 line — tactical action grounded in the Trust Log pattern (per the non-moralizing stance in `references/vibe-check-rubric.md`)
- **Next Step**: 1 line — if any downstream action is implied (prep for a specific meeting, document a prior commitment, raise with manager)

### Output Schema

```markdown
## Vibe Check — [Name] ([ID]) — [date]

- Trust (them→you): [Low / Med / High] [arrow if changed this run]
- Trust (you→them): [Low / Med / High] [arrow if changed this run]
- Last Interaction: [date] ([N] days ago)

### Trust Log (last 3 events)
- [date] [direction] [↑/↓/→] — [reason]
- [date] [direction] [↑/↓/→] — [reason]
- [date] [direction] [↑/↓/→] — [reason]

### Interpretation
[pattern + tactical implication]

**Priority Move**: [tactical action]. **Next Step**: [downstream action if any].
```

## When to use

- Someone's feedback feels unreliable and you want to check whether the pattern is in the data or just vibes
- You missed a commitment and want to log it before you forget
- Before `/prep-1-1` or `/prep-stakeholder` for someone whose trust state you want to re-read
- Before `/review-progress` to see trust red flags that will show up in the Trust Status section
- Session start if `/status` surfaces a trust-related flag

## Boundaries

- `/vibe-check` is for **your** record of these relationships. It is **not** a performance evaluation of the stakeholder. It never leaves `coaching_state.md` (gitignored) and never exports to `interview-coach-skill` or anywhere else.
- Trust state does **not** affect Momentum Score, Trajectory Call, or any performance measure's maturity. Those are performance signal; this is relationship signal. See `references/rubric-performance.md` for the Momentum formula — Trust is not an input.
- The skill does not advise "try to build the relationship" by default (see non-moralizing stance in `references/vibe-check-rubric.md`). Low trust is a coaching signal, not a soft-skills nag.
