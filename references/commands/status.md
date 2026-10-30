# status — Session-start summary

Prescriptive, state-aware greeting. Runs automatically at session start when `coaching_state.md` exists. Also runnable explicitly via `/status`.

## Flow

### Step 1 — Read state

Load `coaching_state.md`. Run Schema Migration Check (see `references/schema-migration.md`). Check archival thresholds (see `references/archival-rules.md`).

### Step 2 — Compose the greeting

Follow this skeleton:

> Welcome back, [Name]. Last session we worked on [X]. Current state: [Y]. Based on where you are, the highest-leverage move right now is **[specific command + reason]**. Want to start there, or tell me what you'd rather work on.

Where:

- **[X]**: the Key Outcome of the most recent Session Log row.
- **[Y]**: one line summarizing current trajectory. Pull from the latest Progress Snapshot if any: "Last snapshot: On Track, Momentum 68 (up from 61 three weeks ago). Measure M003 declined — your Meeting Log shows no action yet." If no Progress Snapshot exists: "No Progress Snapshot yet — [N] artifacts logged, [M] measures tracked." If the user is ramping (Ramp Period in `references/rubric-performance.md`), lead with ramp position instead: "Day [Tenure] of your [Ramp length]-day ramp — [N] checkpoints met, next due: [R#] by day [N]."
- **[command + reason]**: computed via the recommendation logic below.

### Step 3 — Recommendation logic

Check in this order; use the first match:

1. **Stale Scheduled Meeting Prep** (Meeting Preps row with `Status: Scheduled` and Date more than 7 days ago) → surface it: "MTG### for your [1:1 / meeting] with [names] is [N] days old. Did the meeting happen? Run `/log-1-1` or `/log-stakeholder-meeting` to debrief, or discard the prep."
2. **Scheduled Meeting Prep within the next 7 days with no artifact ingested** AND the user mentioned the meeting in a recent session → gentle nudge: "Prep MTG### for [date] — ready to drop in the deck/doc, or want to run the prep again with fresh state?"
3. **Review window < 14 days away** AND no `/review-progress` run in the last 7 days → `/review-progress` ("your review is in [N] days — let's get a current trajectory read").
4. **Feedback floor active** (3+ recent Critical feedback on High-weight measures) → surface it: "Your Feedback Stream shows [N] Critical signals in 30 days on High-weight measures. We should talk about M###. Run `/review-progress` to see the Momentum impact, or `/log-1-1` if you have a 1:1 coming up to address it."
5. **Overdue commitments** (Commitments rows where Due is in the past and Status is Open / Blocked / Renegotiated) → `/commitment --overdue` ("You have [N] commitments past Due with Status still Open. Overdue work compounds trust damage; `/commitment --overdue` to review."). Priority: high — missed deadlines are the loudest trust signal.
6. **Freshly-missed commitments** (Due passed within last 7 days AND Status still Open) → `/commitment <C###>` ("[C###] was due [date]. Was it completed, missed, or renegotiated? Update it before the next 1:1 or stakeholder meeting with [owed-to]."). Narrower than #5; fires when there's a specific individual commitment to resolve.
7. **Blocked-stale commitments** (Status: Blocked AND Last Updated > 14 days ago) → `/commitment <C###>` ("[C###] has been Blocked for [N] days with no status update. Either unblock, renegotiate, or cancel.").
7b. **Ramp checkpoint slipping or missed** (user is ramping; a checkpoint is past its due day and unmet) → the command that closes it: R1 / R8 → `/prep-1-1`; R2 → `/log-1-1` after asking the manager; R3 → `/ingest-leveling` (or `/log-company-goals` first if Company Context is TBD); R4 → `/ingest-stakeholder`; R5-R7 → `/log-artifact`. ("[R#] — [checkpoint] — was due day [N]; you're on day [Tenure]. [command] closes it.") Name only the earliest-due one.
8. **Low-trust signal source on a High-weight measure** → `/vibe-check <id>` ("[name] is the signal source on M### (High weight) and at Low trust. Trust Log shows [pattern]. Prep extra evidence before next review."). Trust does not affect Momentum directly — this is a parallel career-progression risk.
9. **Missed-commitments pattern** (2+ you→them `missed commitment` or `dropped thread` events on anyone in last 60 days) → `/vibe-check <id>` ("[N] missed commitments logged with [name] in 60 days. Worth reviewing the Trust Log before your next interaction.").
10. **Monthly rhythm due** — no `monthly-debrief` for last month and it's days 1-7 → `/monthly debrief` ("Last month hasn't been closed out — the debrief drafts a recap from your weekly recaps for you to edit."); or no `monthly-pre` this month and it's days 1-7 → `/monthly pre`; or last 5 days of the month with no `monthly-debrief` yet → `/monthly debrief`. Same skip condition as the weekly rule below.
11. **Weekly rhythm due** — no `weekly-pre` in Rhythms for the current week (Mon-Sun) and it's Mon/Tue → `/weekly pre`; or no `weekly-debrief` for the current week and it's Thu-Sun → `/weekly debrief` ("Haven't closed out the week yet — `/weekly debrief` drafts a recap from your dailies for you to edit, then runs your progress read and walks open commitments."). Skip if the user has never run a rhythm and Session Log shows they prefer direct commands (see Coaching Notes).
12. **Company Context > 90 days stale** → `/log-company-goals` ("your Company Context was last refreshed [date] — if OKRs rolled over, Priority Weights are off").
13. **Pending `non-quant-reason` artifacts with a `follow up by` date in the past** → `/log-artifact --improve A###` for each, with the specific follow-up measurement to attach.
14. **Over-assessed measures from last `/review-measures`** → `/review-measures` to address.
15. **No Performance Measures Tracker yet** → `/ingest-leveling <path>` ("you haven't ingested your leveling doc yet").
16. **No artifact in last 30 days** → `/daily debrief` if it's after 12:00, else `/weekly debrief` ("you haven't logged work in a month — the debrief will pull out what shipped"). Use `/log-artifact` directly only if the user has already named the thing.
17. **New Meeting Log entry needed** (last Meeting Log row is > 2 weeks ago) → `/log-1-1` ("worth capturing anything from recent 1:1s") — if there's an upcoming scheduled 1:1, suggest `/prep-1-1` instead.
18. Otherwise → the daily rhythm for the time of day: `/daily pre` (no `daily-pre` today, before 17:00) or `/daily debrief` ("Nothing urgent flagging. `/daily pre` takes about a minute and sets up the day — or tell me what's on your mind.").

### Step 4 — Timeline-aware adjustments

If `Review cycle timeline` has passed in the past, proactively ask: "Your review cycle timeline was set to [date], which has passed. Did it happen? How'd it go? Should we log the outcome and set a new review window?"

### Step 4b — One-time schema migration prompts

If a Coaching Note from the schema migration flagged non-canonical Relationship Type values (see `references/schema-migration.md`), surface a one-time recategorize prompt:
> "I remapped Relationship Type values on first read. [N] stakeholders have `Peer (<original>)` placeholders. Want to walk through recategorizing them now, or defer?"

Clear the Coaching Note after the user accepts or defers.

### Step 5 — Session count & meta-check

If `Session Log` row count is a multiple of 3, include a one-line meta-check at the end of the greeting: "This is session [N] — is the coaching useful? Anything you'd change in how we're working?"

### Step 6 — Optional append

Only append a Session Log row for `/status` if the user runs it explicitly as a command. Automatic session-start status reads don't log — otherwise every open-chat creates noise.

## Output Schema

Keep this short — one paragraph plus the recommended-next line.

```markdown
Welcome back, [Name]. Last session: [X]. Current state: [one-line trajectory read]. [Any floor-level or stale-check callouts.]

**Highest-leverage move**: `[command]` — [reason].

[Optional one-line meta-check if session count % 3 == 0.]
```
