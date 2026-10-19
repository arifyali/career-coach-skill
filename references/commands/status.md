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
- **[Y]**: one line summarizing current trajectory. Pull from the latest Progress Snapshot if any: "Last snapshot: On Track, Momentum 68 (up from 61 three weeks ago). Measure M003 declined — your 1:1 Log shows no action yet." If no Progress Snapshot exists: "No Progress Snapshot yet — [N] artifacts logged, [M] measures tracked."
- **[command + reason]**: computed via the recommendation logic below.

### Step 3 — Recommendation logic

Check in this order; use the first match:

1. **Review window < 14 days away** AND no `/review-progress` run in the last 7 days → `/review-progress` ("your review is in [N] days — let's get a current trajectory read").
2. **Feedback floor active** (3+ recent Critical feedback on High-weight measures) → surface it: "Your Feedback Stream shows [N] Critical signals in 30 days on High-weight measures. We should talk about M###. Run `/review-progress` to see the Momentum impact, or `/log-1-1` if you have a 1:1 coming up to address it."
3. **Company Context > 90 days stale** → `/log-company-goals` ("your Company Context was last refreshed [date] — if OKRs rolled over, Priority Weights are off").
4. **Pending `non-quant-reason` artifacts with a `follow up by` date in the past** → `/log-artifact --improve A###` for each, with the specific follow-up measurement to attach.
5. **Over-assessed measures from last `/review-measures`** → `/review-measures` to address.
6. **No Performance Measures Tracker yet** → `/ingest-leveling <path>` ("you haven't ingested your leveling doc yet").
7. **No artifact in last 30 days** → `/log-artifact` ("you haven't logged work in a month — let's capture what shipped").
8. **New 1:1 Log entry needed** (last 1:1 Log row is > 2 weeks ago) → `/log-1-1` ("worth capturing anything from recent 1:1s").
9. Otherwise → prompt the user: "Nothing urgent flagging. What's on your mind?"

### Step 4 — Timeline-aware adjustments

If `Review cycle timeline` has passed in the past, proactively ask: "Your review cycle timeline was set to [date], which has passed. Did it happen? How'd it go? Should we log the outcome and set a new review window?"

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
