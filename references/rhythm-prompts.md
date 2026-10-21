# Rhythm Prompts

Central reference for the structured prompts used by `/daily`, `/weekly`, and `/monthly`. Also defines the recap roll-up protocol (how weekly and monthly debriefs build on the layer below), response-detection rules (how to spot an artifact / feedback / commitment candidate in a free-text answer and route it to the right capture flow) and the response-blueprint overrides for rhythm commands.

Read this before `/daily`, `/weekly`, or `/monthly`.

## Prompt sets

### Daily pre-brief — 3 prompts

Fire one at a time per the non-negotiable sequencing rule.

1. **Intention**: "What's the one thing you want to be true at end of day?"
2. **Biggest risk**: "What's the biggest risk today — a decision, a dependency, a conversation you've been putting off?"
3. **Raise with**: "Who do you need to raise what with today?" (The skill pre-surfaces candidates from recent Trust Log drift, overdue commitments, pending Feedback Stream entries not yet linked to a Meeting Log row.)

### Daily debrief — 3 prompts

1. **Shipped**: "What shipped today?" (If the user names a shippable outcome with any measurable Y — adoption, speed, quality, dollars — treat as artifact candidate per response-detection rules.)
2. **Unresolved**: "What's unresolved — rolling into tomorrow?"
3. **Surprises**: "Anyone or anything surprise you today?" (Trust / feedback signal; detect routing.)

### Weekly pre-brief — 4 prompts

1. **Outcome**: "What's the biggest outcome you want from this week?"
2. **Biggest risk**: "What's the biggest risk — a dependency, a stakeholder, a timeline?"
3. **Commitments lock**: "Which commitments do you need to lock in or renegotiate before Friday?" (Pre-surface candidates from Commitments table — Open rows with Due this week or `not agreed`.)
4. **Meetings worth prepping**: "Any 1:1s or stakeholder meetings worth prepping now?" (Surface Meeting Preps for the next 7 days that have `Status: Scheduled` and no linked artifact.)

### Weekly debrief — 3 prompts (after the recap is confirmed and `/review-progress` runs)

1. **Headline**: "Of what's in the recap, what's the single most important thing that shipped?"
2. **Career-goal movement**: "What moved on your short-term Career Goal?" (Reference the latest Progress Snapshot's Trajectory Call for grounding.)
3. **Shifts worth noting**: "Anything in the recap that's a priority / relationship / trust shift worth capturing before it fades?"

Order matters: the user has just confirmed the recap and seen Momentum math + Trust Status from `/review-progress`; these prompts translate that into intention-level reflection. Skip or convert to a confirm question any prompt the recap already answers.

### Monthly pre-brief — 3 prompts

1. **Focus**: "What's the one thing this month should move toward your career goal?" (Offer Measures to watch and last month's `Carry forward`.)
2. **Evidence by month-end**: "What evidence do you want to have by the end of the month — a shipped artifact, a piece of feedback, a conversation?"
3. **Relationship to invest in**: "Whose read on your work matters most this month, and what do they need to see?"

### Monthly debrief — 2 prompts (after the recap is confirmed and `/review-measures` runs)

1. **Against the goal**: "Looking at that recap, did this month move you toward [short-term Career Goal]? What's the honest read?"
2. **Change for next month**: "What do you want to do differently next month — keep, stop, or start?"

## Recap roll-up protocol

Weekly and monthly debriefs don't start from a blank page. Each builds a draft recap from the layer below, then works with the user to correct and extend it. The confirmed recap is stored on the debrief entry and becomes the base layer for the next level up.

| Debrief | Base layer | Also reads |
|---|---|---|
| `/weekly debrief` | The week's `daily-pre` / `daily-debrief` entries | `weekly-pre` entry; captures dated in the week (A###, F###, C### status changes, Meeting Log rows) |
| `/monthly debrief` | The month's `weekly-debrief` `Recap` lines (daily entries for weeks without one) | `monthly-pre` entry; captures dated in the month; Progress Snapshots in the month |

Steps, one question at a time:

1. **Gather** the base layer and captures for the period.
2. **Draft** a short recap (weekly: 3-5 lines; monthly: 4-6 lines). Cite IDs for anything captured. Measure it against the period's pre-brief (Outcome / Focus) — "You set out to X; Y happened."
3. **Show and build**: present the draft and ask "What's wrong or missing?" Then invite any recap the user already wrote elsewhere (a status update, a Slack note, a self-review draft) and fold it in. This replaces a separate paragraph-paste step.
4. **Route** anything new that reads as an artifact / feedback / commitment through response-detection below; captured IDs go into the recap.
5. **Confirm** the final text with the user before writing it.

Rules:
- **Never invent.** If days or weeks have no entries and no captures, name the gap ("Nothing logged Wed-Thu — anything from those days?") instead of smoothing over it.
- **The user's words win.** When the user corrects the draft, take their framing; don't re-argue it in the recap. Disagreement on trajectory belongs in the prompts and coaching read, not in the recap text.
- **Carry forward explicitly.** Unresolved items from the recap feed the next pre-brief (weekly → next `weekly-pre` dashboard; monthly → `Carry forward` → next `monthly-pre`).

## Response-detection rules

When a user's answer to a structured prompt contains a signal of a shippable outcome, feedback event, or accepted commitment, route to the appropriate capture flow. The user walks each routed candidate through the destination command's acceptance step; nothing writes to state until the user confirms.

### Shippable outcome signals → `/log-artifact` candidate

Trigger: the user names an action verb (shipped, launched, released, deployed, published, delivered, sent, closed) + a specific object + any quantifiable surrogate (number of users, revenue, latency, accuracy, adoption, hours saved).

Pattern: `"[shipped] [thing] [with Y]"` → propose Artifact candidate with X (what), proposed Y (number), proposed Z (how, if the user volunteered it).

If no quantifiable Y is present, don't auto-promote — propose the candidate as "needs Y" and surface the `non-quant-reason` guardrail from `references/xyz-artifact-guide.md`. The quantified-Y rule still applies inside rhythm flows.

### Feedback signals → `/log-feedback` candidate

Trigger: the user attributes a statement or impression to a named person or role (manager, stakeholder K###, peer, review doc). Examples:
- "My manager said..." / "Priya pushed back on..." / "The design review flagged..."
- "I got the sense that..." (implicit signal — propose as Mixed sentiment)

Pattern: `"[source] [said / asked / flagged] [content]"` → propose Feedback Stream candidate with Source resolved via FK pattern (per `references/commands/log-feedback.md`), Sentiment proposed, Verbatim/Paraphrase pre-filled.

If the user names a stakeholder/manager without quoting content ("I had a tense sync with K004"), surface as a Trust-signal candidate instead — route to `/vibe-check <id>` rather than `/log-feedback`.

### Commitment signals → `/commitment add` candidate

Use the extraction heuristic in `references/commitment-schema.md`. Strong signals: "I told X I'd...", "I promised...", "I'll have X by Y." Weak signals ("I should probably...") don't auto-promote.

In rhythm flows, commitment candidates often emerge from the "Who do you need to raise what with" (daily pre) or "Unresolved" (daily debrief) prompts. Explicitly ask: "Is any of this a formal commitment you've accepted?"

### Trust signals (not routed to a dedicated capture)

Trust events in rhythm flows surface a *suggestion* to run `/vibe-check <id>`, but do NOT auto-propose a Trust Log entry. Per the opt-in design in `references/vibe-check-rubric.md`, trust capture stays explicit. Example output:

> "The way you described K004's goalpost shift today sounds like a Trust (them→you) ↓ event. Want to run `/vibe-check K004` to log it, or skip?"

## Response-blueprint override for rhythm commands

Rhythm commands are rituals, not deep workflows. The standard response blueprint (`What I Heard → What Is Working → Gaps To Close → Priority Move → Next Step`) is too heavy. Use this tighter blueprint:

- **Dashboard** (pre-brief only) — read-only state surface
- **Recap** (weekly / monthly debrief only) — the confirmed roll-up, built with the user
- **Prompts** — walked one at a time
- **Captures summary** — one line listing what landed in which tables
- **Priority Move** — single most leveraged next action (only if a clear one emerged from the prompts; otherwise skip)

Keep the output short. If the daily pre takes more than ~90 seconds of real time, the ritual is too heavy and the user will stop running it.

## Rhythms entry format

Each rhythm run (`/daily`, `/weekly`, `/monthly`) writes an entry to the `## Rhythms` section in `coaching_state.md`. Format defined in `references/coaching-state-schema.md`. Short bullets + a `Captures:` summary line.

Example daily-pre entry written after a hypothetical run:

```markdown
#### daily-pre (08:30)
- Intention: Finish the model monitoring design doc draft.
- Biggest risk: Haven't talked to K002 about data access.
- Raise with: K002 (data access ask), MGR (scope-change update).
- Captures: 1 commitment (C004).
```

The `Captures:` line is the audit trail. If the user is scanning rhythm history for "what did I promise last Tuesday morning," they can see entries + follow refs into the Commitments / Artifact Library / Feedback Stream tables.

## When routing fires inside a rhythm flow

- **Shippable outcome in a response** → run `/log-artifact` candidate flow inline. User walks the standard XYZ capture (one-at-a-time) with values pre-filled from what they just said. Writes `A###` on acceptance; appends to Captures line.
- **Feedback signal in a response** → run `/log-feedback` candidate flow inline. User walks Source resolution + Sentiment + linked measure. Writes Feedback Stream row on acceptance.
- **Commitment signal in a response** → run `/commitment add` candidate flow inline per the extraction hook pattern already defined in debrief flows. Writes `C###` on acceptance.
- **Trust signal in a response** → surface as a suggestion with `/vibe-check <id>` as the explicit next action. Do NOT run inline — user decides whether to run `/vibe-check` after the rhythm closes.

All inline routed captures append to the rhythm entry's `Captures:` summary.

## Non-moralizing stance

Mirrors `references/vibe-check-rubric.md` and `references/commitment-schema.md`. If the user runs `/daily debrief` and reports "I didn't do anything today" or skips most prompts, the skill does **not** nag or push for more. The ritual is user-driven; the capture is best-effort.

What the skill does:
- Capture whatever the user offers.
- Note empty entries in Rhythms (useful signal across weeks even if a single day is empty).
- At the weekly debrief, if 3+ daily entries were empty, surface the pattern once without judgment: "3 of your daily debriefs this week were empty. Not a problem; just naming it in case the daily ritual isn't fitting." Let the user decide what to do.
