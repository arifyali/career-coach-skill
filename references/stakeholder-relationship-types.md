# Stakeholder Relationship Types

The Relationship Type on each Stakeholder Backgrounds row drives how the skill preps meetings with that stakeholder and what drift to look for at debrief. Use the canonical enum below — don't invent new types ad hoc.

## Canonical enum

- `Partner` — collaborator at roughly peer power; mutual delivery dependencies. Trust is built on reliability of commitments and shared ownership.
- `Senior Partner` — collaborator above the user's level; evaluating + mentoring surface. Decisions and tradeoff framing matter more than status detail.
- `Upstream` — supplies inputs the user depends on (data producer, infra owner, PM feeding roadmap). The user is a consumer of their work.
- `Downstream` — consumes outputs the user produces (consumer team, exec dashboard, downstream model, product surface). The user is their supplier.
- `Customer` — external or internal recipient the user is measured by on satisfaction / adoption. Different from `Downstream` in that the user is accountable for *their* outcome, not just the handoff.
- `Reviewer` — reviews the user's work formally (approver, calibration committee, review-cycle stakeholder, architecture review board). Interactions skew evaluative.
- `Peer` — same team, same level, horizontal. Different from `Partner` in that there's no explicit delivery dependency — the relationship is collegial.

Multiple types allowed per stakeholder. Common combinations:
- `Partner + Reviewer` — a cross-team collaborator who also sits on your calibration committee.
- `Upstream + Senior Partner` — the senior engineer whose infra decisions shape your work.
- `Downstream + Customer` — the consumer team whose success is your success metric.

Record multiple types comma-separated: `Partner, Reviewer`.

## How Relationship Type steers prep

### Agenda ordering (single-participant meeting)

| Type | Lead with | Why |
|------|-----------|-----|
| `Customer` | their asks + user's response to prior asks | preserves their time; shows the user is listening |
| `Senior Partner` | decisions needed, time-boxed, 1-line context | respects their altitude; forces user to come with a crisp ask |
| `Reviewer` | evidence against measures | the meeting is implicitly evaluative; frame it that way |
| `Upstream` | blockers, SLAs, prioritization | user depends on their output; lead with what's at risk |
| `Downstream` | roadmap signals, early warning of breaking changes | they're consumers; give them lead time |
| `Partner` | mutual coordination, shared commitments | symmetric relationship; split the agenda |
| `Peer` | shared context, mutual help, informal signal | low-ceremony; keep it light |

### Anticipated questions

| Type | Likely to ask | Prep implication |
|------|---------------|------------------|
| `Reviewer` | "How did you measure Y?" / "What's the evidence?" | prep concrete artifact IDs + metrics for every claim |
| `Customer` | "When does X ship?" / "Does X solve Z?" | prep ship dates + outcome mapping, not internal details |
| `Senior Partner` | "What's the tradeoff?" / "Why not Y instead?" | prep decision frames, options considered, cost structure |
| `Upstream` | "Can your team absorb X by [date]?" / "Priority between A and B?" | prep capacity + priority rank |
| `Downstream` | "What breaks when X changes?" / "Can we pin on version Y?" | prep breaking-change surface + migration path |
| `Partner` | "Who owns Y?" / "What's your commit?" | prep crisp commitments + handoff points |
| `Peer` | "How did you handle X?" / "Any gotchas?" | prep candid signal; this is where war stories go |

### Questions-to-ask

Biased toward filling that type's typical Open Gaps:
- `Partner` / `Peer`: alignment on scope, mutual commitments, blockers each sees in the other
- `Reviewer`: calibration standards ("what would move me from Meeting to Exceeding?")
- `Customer`: satisfaction + usage patterns + unmet needs
- `Upstream`: capacity signals, roadmap for the inputs user depends on
- `Downstream`: usage of user's outputs, pain points, drift from spec
- `Senior Partner`: direction / strategy / where the user should spend attention next

## How Relationship Type steers debrief

At `/log-1-1` or `/log-stakeholder-meeting` time, after the debrief, run a per-participant drift check. For each `K###` in the prep's Participant IDs:

- **Priorities drift**: snapshot's top-3 Priorities vs. what the debrief surfaced. If different, offer to update Stakeholder Backgrounds.
- **Relationship Type drift**: did the interaction feel like a different type? A `Partner` who is suddenly asking only evaluative questions may be functioning more as a `Reviewer`. A `Senior Partner` who's getting into weeds may be functioning as `Partner` now. Offer to reclassify.
- **Open Gaps**: closed any? Opened any new ones?

For `MGR` / `SKIP`, the drift check runs against Manager Background, not Stakeholder Backgrounds.

## Multi-participant meetings

When multiple relationship types are in the room:
- **Agenda ordering**: use "highest-authority-first for decisions, owner-first for status." Put items needing `MGR` or `Senior Partner` input early; put collaboration/handoff items with `Partner` / `Peer` after. Name primary participant per item ("Decision needed from [manager]:" / "Status update to [customer]:").
- **Anticipated questions**: group by participant. Don't blend. "Expect [name] (Reviewer) to ask: ... Expect [name] (Partner) to ask: ..."
- **Questions-to-ask**: group by participant. Explicit per-person — avoids the "ask the room" ambiguity.

## Legacy value migration

v1 of the skill used informal strings like `collaborator`, `blocker`, `sponsor`. On first read of a legacy state file, `references/schema-migration.md` runs a best-effort mapping:

| v1 string | Canonical v2 |
|-----------|--------------|
| `collaborator` | `Partner` |
| `sponsor` | `Senior Partner` |
| `blocker` | `Upstream` |
| `customer` | `Customer` |
| `peer` | `Peer` |

Unmatched values become `Peer (<original>)` and `/status` surfaces a one-time recategorize prompt.
