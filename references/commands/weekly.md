# weekly — Weekly brief and debrief

Weekly ritual that zooms out from `/daily`. Pre-brief previews the week ahead (meetings, commitments due, Progress Snapshot age) and picks up where last week's recap left off; debrief builds a week recap on top of the week's daily entries, works with the user to correct and extend it, then wraps `/review-progress` and adds bulk commitment flipping and trust review. The confirmed weekly recap is what `/monthly debrief` builds on.

Prompts, routing rules, and the recap roll-up protocol live in `references/rhythm-prompts.md` — read that reference before this file. For debrief, also read `references/commands/review-progress.md` since `/weekly debrief` auto-invokes it.

## Invocation

```
/weekly                  # infer pre vs. debrief from day of week
/weekly pre              # explicit Monday-ish brief
/weekly debrief          # explicit Friday-ish debrief
```

### Day-of-week heuristic (implicit mode)

- Monday / Tuesday → `pre`
- Thursday / Friday → `debrief`
- Wednesday → ambiguous; ask "pre or debrief?"
- Weekend → ambiguous; ask (Saturday leans debrief for the week just-ended; Sunday leans pre for the week ahead)

If a `weekly-pre` entry already exists for the current week (Mon-Sun), prefer `debrief` on implicit calls — user did pre, is now debriefing.

---

## Pre-brief flow (`/weekly pre`)

### Step 1 — Dashboard

Read-only surface of the next 7 days:

- **Last week's recap**: the `Recap` line and `Priority Move for next week` from the most recent `weekly-debrief`. If there's a `monthly-pre` for the current month, also show its `Focus` line. If neither exists, skip.
- **Meetings this week**: Meeting Preps rows with `Date` in next 7 days AND `Status: Scheduled`. For each: `MTG### — [names] on [date]`.
- **Commitments due this week**: Commitments rows with `Status: Open` AND `Due` in next 7 days (or `not agreed` with Last Updated > 7 days ago — those need attention too).
- **Progress Snapshot age**: if last Progress Snapshot was > 7 days ago, surface: "Last snapshot was [date] ([N] days ago). Consider running `/review-progress` during this week's debrief."
- **Trust Log drift from the previous week**: scan Trust Log entries in the last 7 days. If 2+ events on anyone (any direction), surface: "Trust movement last week: [N] events on [name(s)] — worth revisiting at the debrief."

### Step 2 — Structured prompts (4)

Per `references/rhythm-prompts.md` — Weekly pre-brief set. One at a time:

1. **Outcome**: "What's the biggest outcome you want from this week?" (If last week's recap carried a Priority Move or the month has a Focus, offer them: "Last week you set [X] as the move for this week — still the one, or has it changed?")
2. **Biggest risk**: "What's the biggest risk — a dependency, a stakeholder, a timeline?"
3. **Commitments lock**: "Which commitments do you need to lock in or renegotiate before Friday?" (Skill pre-surfaces Open rows with Due this week + rows with `not agreed` Due that are > 7 days old.)
4. **Meetings worth prepping**: "Any 1:1s or stakeholder meetings worth prepping now?" (Skill pre-surfaces Meeting Preps for next 7 days that have `Status: Scheduled` and no artifact linked — prime for `/prep-1-1` or `/prep-stakeholder`.)

Apply response-detection rules per `rhythm-prompts.md`:
- Commitment signal on #3 → inline `/commitment add` or `/commitment <C###> renegotiate` candidate flow
- Meeting prep intent on #4 → surface `/prep-1-1` or `/prep-stakeholder <id>` as suggested next command (don't auto-launch — user picks)

### Step 3 — Optional paragraph-paste

Short — weekly pre is lighter on capture. "Anything else to capture before the week starts? (Short paragraph is fine.)"

### Step 4 — Write Rhythms entry + Session Log

```markdown
### [today's date]
#### weekly-pre ([day of week] [HH:MM])
- Outcome: [response]
- Biggest risk: [response]
- Commitments to lock: [response, with C### refs where mentioned]
- Prep queue: [response, with MTG### refs or stakeholder names]
- Captures: [summary]
```

Session Log row: `weekly-pre | [capture summary]`.

### Step 5 — Priority Move (optional)

If one meeting prep or commitment lock stands out as the week-defining action, emit:

> **Priority Move for the week**: [specific command + reason].

Skip if everything's equally weighted.

---

## Debrief flow (`/weekly debrief`)

### Step 1 — Build the week recap

Follow the recap roll-up protocol in `references/rhythm-prompts.md`, with the week (Mon-Sun) as the period and daily entries as the base layer:

1. **Gather**: the week's `daily-pre` / `daily-debrief` entries (Shipped, Unresolved, Surprises especially), the `weekly-pre` entry, and the week's captures (A###, F###, C### status changes, Meeting Log rows).
2. **Draft**: 3-5 lines — what shipped (by ID where captured), what moved with whom, what's still open, and how the week landed against the `weekly-pre` Outcome.
3. **Show and build**: present the draft, then one question at a time: "What's wrong or missing?" → "Did you write a recap anywhere else this week — a status update, a Slack note to your manager? Paste it and I'll fold it in." Merge what the user gives; anything that reads as a new artifact / feedback / commitment routes to its capture flow per response-detection rules.
4. **Confirm**: show the final recap and get a `yes` / edit before writing.

If the week has no daily entries, say so and build the draft from captures alone; if there are no captures either, ask the user for the week in a paragraph and draft from that.

### Step 2 — Auto-invoke `/review-progress`

Run the full `/review-progress` flow inline per `references/commands/review-progress.md`. The Momentum math, Trajectory Call, and Trust Status read all fire exactly as if `/review-progress` were run standalone. A Progress Snapshot row writes.

**Do not** re-implement the Momentum math here. A ramping user with an empty Tracker still gets the Ramp Pace read (see the ramping exception in `review-progress.md`). If `/review-progress` can't run otherwise (no Performance Measures Tracker, or no short-term Career Goal), surface the error and continue with the weekly-specific steps below — the ritual still has value even without the Momentum read.

### Step 3 — Weekly-specific structured prompts (3)

Per `references/rhythm-prompts.md` — Weekly debrief set. One at a time, after `/review-progress` returns. These build on the confirmed recap — don't re-ask what it already answers:

1. **Headline**: "Of what's in the recap, what's the single most important thing that shipped?" (If the recap makes it obvious, propose it and ask to confirm instead.)
2. **Career-goal movement**: "What moved on your short-term Career Goal?" (Reference the Trajectory Call just surfaced for grounding.)
3. **Shifts worth noting**: "Anything in the recap that's a priority / relationship / trust shift worth capturing before it fades?"

Apply response-detection rules:
- Shippable outcome on #1 not already captured → inline `/log-artifact` candidate (quantified-Y guardrail; this is often where the week's headline artifact gets logged)
- Trust signal on #3 → surface `/vibe-check <id>` suggestion
- Priority / company-context shift on #3 → surface `/log-company-goals` suggestion

### Step 4 — Bulk commitment status flip pass

Read Commitments rows where `Status: Open OR Blocked OR Renegotiated` AND (`Due in last 7 days` OR `Due in next 7 days` OR `Last Updated > 7 days ago`).

For each row:
> "C### — [commitment one-line] — Status: [state] — Due: [date]. Flip to: `complete` / `block` / `renegotiate` / `cancel` / `unchanged` / `skip`?"

Each non-unchanged flip runs the full status-flip sub-flow per `references/commands/commitment.md`, including Trust Log coupling. Bulk means walking each in sequence — not batch-silent-write.

If no commitments match, skip Step 4 silently.

### Step 5 — Trust review pass

Read Trust Log entries from the last 7 days across Manager Background and all Stakeholder Details blocks. Surface:

```
- [name] (K### or MGR): [N] events — [direction summary, e.g., "2 ↓ them→you"]
```

For each person with changes:
> "Trust Log says [name] moved [direction] this week after [event summary]. Confirm, revise, or skip?"

- `confirm` → leave current Trust score as-is
- `revise` → launch `/vibe-check <id>` inline
- `skip` → move on

If no Trust Log activity in last 7 days, skip Step 5 silently.

### Step 6 — Write Rhythms entry + Session Log

```markdown
#### weekly-debrief ([day] [HH:MM])
- Recap: [confirmed recap, 3-5 lines]
- Progress Snapshot: Momentum [N] ([delta] from last week), Trajectory [X]
- Headline: [response, with A### ref if captured]
- Career-goal movement: [response]
- Shifts: [response]
- Commitment flips: [summary like "C001 → Completed, C004 → Renegotiated, 3 unchanged"]
- Trust review: [summary like "K004 confirmed Low; K001 confirmed High"]
- Captures: [summary of all writes this run]
```

Session Log row: `weekly-debrief | Momentum [N] delta [+/-N], [N] flips, [capture summary]`.

### Step 7 — Priority Move for next week

Emit at the end:

> **Priority Move for next week**: [specific action grounded in Momentum delta + Trust shifts + Commitment flips].

The weekly debrief is a natural place to force a Priority Move — the user has just reflected on the full week; a next-week recommendation is high-signal.

### Step 8 — Empty-daily-pattern check

If 3+ `daily-pre` or `daily-debrief` entries in the last 7 days were empty (user ran the command, skipped most prompts, Captures was none), surface a single line:

> "3 of your daily rhythm entries this week were empty. Not a problem; just naming it in case the daily ritual isn't fitting your workflow. Could skip to just `/weekly` and run `/daily` only when something's active."

Per the non-moralizing stance. One-time surface per week; don't repeat it.

---

## Output schema

### Pre-brief

```markdown
## Weekly Brief — Week of [Monday date]

### Dashboard
- Last week: [recap one-liner + carried Priority Move, or "no recap"]
- Meetings this week: [N]
- Commitments due this week: [N]
- Progress Snapshot age: [N] days
- Trust activity last week: [N] events

### Prompts
- Outcome: [response]
- Biggest risk: [response]
- Commitments to lock: [response]
- Prep queue: [response]

### Captures
[list of A###, C###, F### written, or "none"]

[optional **Priority Move for the week**: ...]
```

### Debrief

```markdown
## Weekly Debrief — Week of [Monday date]

### Recap
[confirmed recap]

[full /review-progress output embedded here as a sub-section]

### Weekly reflection
- Headline: [response]
- Career-goal movement: [response]
- Shifts: [response]

### Commitment flips
- C###: [prior] → [new]
- C###: unchanged
- ...

### Trust review
- [name]: [confirmed / revised / skipped]
- ...

### Captures
[list of writes this run]

**Priority Move for next week**: [specific action].

[optional empty-daily-pattern note]
```

## Boundaries

- The recap draft is a starting point, never written without the user's confirmation. Don't invent outcomes for days with no entries — name the gap.
- `/weekly debrief` ALWAYS attempts `/review-progress` after the recap. If it fails (missing prerequisites), the weekly flow continues without Momentum data but notes the gap.
- Prompts fire one at a time, including inside the auto-invoked `/review-progress`.
- Bulk status flipping walks each commitment individually. No batch-silent-write — Trust Log coupling fires on each.
- Rhythms entries never export via `/export-artifacts`.
- Pattern surfacing (3+ empty daily entries) is one-time per week; the skill doesn't nag.
