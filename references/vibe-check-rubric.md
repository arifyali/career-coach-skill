# Vibe-Check Rubric

Trust scoring for managers and stakeholders. Two directions, each with its own score. Captured via `/vibe-check` — opt-in, not inline on `/log-feedback` or meeting debriefs.

## Scale

3-level, mirrors Evidence Strength and Priority Weight patterns:

- **High** — track record of signal reliability (or user-side follow-through) in this direction
- **Medium** — default; neutral; signal not yet differentiated
- **Low** — pattern of poor signal (or missed follow-through) in this direction

Every new person starts at **Medium / Medium**. Decay (see below) returns scores toward Medium over time.

## Two directions

### Trust (them → you)

Should the user trust **this person's** signal?

**Moves it up (↑)**:
- Specific, actionable feedback cited to concrete artifact or outcome
- Consistency: same priorities named across multiple interactions over 60+ days
- Priority stability: when they do shift priorities, they acknowledge the shift and explain why
- Availability when it matters (unblocked a decision, showed up for a critical review)
- Kept commitments they made to the user

**Moves it down (↓)**:
- Vague feedback with no evidence tie ("be more strategic", "show more leadership")
- Goalpost shifts without acknowledgment — asking for X in one interaction, then Y in the next with no explanation
- Contradictions between what they say and what shows up in review docs or calibration
- Unavailable when critical (missed a key review, no-showed on a decision gate)
- Broke commitments to the user without flagging

### Trust (you → them)

Is the user **keeping trust with this person**?

**Moves it up (↑)**:
- Kept your commitments on time
- Responsive on their asks within the agreed cadence
- Proactively flagged risk or scope changes before they became problems
- Delivered what was agreed, including the quiet work no one would have noticed

**Moves it down (↓)**:
- Missed commitments without flagging
- Dropped threads — follow-ups logged but never actioned
- Slow responses on their critical asks
- Surprised them on something they should have known about from you

## Scoring rules

- Each logged event moves trust by **at most one notch** in one direction.
- Movement is bounded by prior state: High can't go higher; Low can't go lower.
- Both directions can regress to Medium then invert direction over multiple events. There's no "memory" of prior extremes beyond the Trust Log.
- Trust events are **always logged with a reason** in the Trust Log — the one-line reason is more valuable than the score itself for later review.

## Decay rule

If `Last Interaction` on a stakeholder (or `Last enriched` on Manager Background) is **more than 90 days ago**, both trust scores decay one notch toward Medium:
- High → Medium
- Low → Medium
- Medium → Medium (no change)

Decay is applied at runtime by `/vibe-check` and `/status`. When applied, write a Trust Log entry: `[date] decay — no interaction in [N] days; [old] → [new]`. The user can log a fresh event in the same session to override the decay if they have signal they haven't captured yet.

Decay is intentionally gentle. A single 90-day gap doesn't flip High to Low — it moves toward neutral. The rubric assumes most relationships that go cold aren't fundamentally broken; they just need re-establishment.

## Non-moralizing stance

**Important**: this skill does not treat "low trust" as a problem the user needs to fix. Not everyone in the workplace is reliable, and not everyone needs to be a friend. The skill's job is to:

1. **Name the pattern directly** — "K004 has shifted goalposts three times in 60 days on M005. Their feedback is unreliable on this measure."
2. **Suggest tactical responses** — document everything in writing, prefer async channels with timestamps, raise with manager if the person is blocking critical work.
3. **Make the career-progression implication visible** — "Low-trust Reviewer on a High-weight measure is a review-cycle risk; prep extra evidence."

What the skill does NOT do by default:

- Advise "try to build the relationship" or "find common ground." Relationship-repair advice is only given if the user explicitly asks for it.
- Pathologize the user's reaction to vague or shifting feedback. Noticing those patterns is the correct response, not an interpersonal failure.
- Attribute low trust to communication style or temperament unless the user has data pointing that way.

If the user asks for relationship-repair advice (`/vibe-check <id>` and says "how do I rebuild this"), provide it — but make it tactical and specific, grounded in the Trust Log, not generic.

## Event-type menu (for `/vibe-check`)

Default event types the command flow offers at the "what happened" step:

**Them → you**
- `solid feedback` → ↑
- `vague feedback` → ↓
- `goalpost shift` → ↓
- `consistency` → ↑ (same priority named over time)
- `inconsistency` → ↓
- `unavailable when it mattered` → ↓
- `showed up when it mattered` → ↑
- `specific event (free text)` → user chooses ↑/↓/→

**You → them**
- `kept commitment` → ↑
- `missed commitment` → ↓
- `proactive risk flag` → ↑
- `dropped thread` → ↓
- `delivered on ask` → ↑
- `slow on critical ask` → ↓
- `specific event (free text)` → user chooses ↑/↓/→

User can always override the default impact and the event type can always be free text.

**Commitment coupling**: `kept commitment`, `missed commitment`, `renegotiated commitment`, and `dropped thread` events can be auto-proposed by `/commitment` status flips — see `references/commitment-schema.md`. In that flow, the Trust Log entry references the `C###` and the user confirms yes/no before the write. The event-type menu above is still used when the user runs `/vibe-check` directly without an underlying commitment record (e.g., logging a verbal promise that was never structured as a Commitment).

## Interpretation lines

After an event is logged (or at `/vibe-check <id>` deep view), the command surfaces a one-line interpretation. Format: pattern + tactical implication.

Examples:
- "K004 them→you at Low after two goalpost shifts in 30 days. Document any new asks in writing; cite the written record if the goalposts shift again."
- "MGR you→them at Low after missing C004 (Q3 doc review, due 2026-09-15) and C007 (migration plan, due 2026-10-05). The miss on C004 is the one to repair first — raise it unprompted at the next 1:1."
- "K001 them→you at High across 4 months with specific feedback consistently. Treat their input as high-signal; weight it more than K004's on overlapping measures."
- "K007 both directions decayed from High to Medium — no interaction in 94 days. Not broken; just cold. Worth a sync if they're a signal source on anything active."

Interpretation is always paired with a concrete tactical next action, never with generic advice. When recent Trust Log events reference specific `C###` IDs (from the commitment coupling), cite them in the interpretation — grounds the pattern in records the user can trace rather than vague "missed commitments."
