# TODO

Working roadmap for `career-coach-skill`. Public — this is where the skill is going, not personal state. Items marked `[in progress]` or `[next up]` are actively being worked. Items marked `[deferred]` are scoped but not scheduled. Items in **Polish & observations** are smaller improvements noted during dogfood.

Status key: `[next up]`, `[in progress]`, `[deferred]`, `[polish]`, `[open question]`.

## Next up

- **[next up] Commitments table + flows.** Dedicated top-level section in `coaching_state.md` with Commitment / Owner / Owed to / Due / Status (Open / Blocked / Completed / Renegotiated / Cancelled) / Source. New `/commit` or `/log-commitment` command plus hooks into existing flows: extract from meeting debriefs, surface outstanding ones at `/prep-*` time, close + capture evidence at completion. Becomes the source of truth for `/vibe-check`'s Trust (you→them) `kept commitment` / `missed commitment` / `dropped thread` events — currently logged ad-hoc. Rule: only accepted commitments go in the table (a coach suggestion isn't a commitment until the user adopts it). Also: paragraph-paste capture — paste a rough weekly recap, skill extracts commitments + feedback + accomplishments without per-item entry. Source: ChatGPT analysis item #1 (highest-priority recommendation).

## Deferred — from the ChatGPT analysis (external review)

- **[deferred] Confirmed manager expectations.** Add a `Confirmed Expectations` sub-block to Manager Background covering the five areas worth clarifying in first 1:1s: Success (what makes 30-90 days good), Ownership (what you own + definition of done), Decision authority (independent vs. discuss-first), Communication (updates + risk-raising), Prioritization & feedback (competing requests, candid feedback). Each item tagged Confirmed / Working hypothesis / Unresolved. Not a formal document to present — accrues naturally over 1:1s. Pre-start useful. Depends on nothing. Source: ChatGPT analysis item #2.

- **[deferred] Interactive conversation rehearsal.** Multi-turn rehearsal command: skill plays a stakeholder using their known Priorities + Relationship Type, user practices answering, skill gives feedback on observable parts (did you state a recommendation, explain the tradeoff, acknowledge their needs, ask for a decision, get defensive). Critical guardrail: **simulated dialogue must never pollute the factual relationship record** — a plausible objection generated during rehearsal cannot become evidence that the manager actually holds that view. One leaky implementation corrupts Trust Log / Meeting Log / Feedback Stream at review time. Build carefully or not at all. Source: ChatGPT analysis item #3.

- **[deferred] Project selection in career-strategy mode.** Introduce a career-strategy top-level mode (new scaffolding — doesn't exist yet). First workflow inside it: compare 2-3 project options across Business value / Ownership / Development / Advancement / Future direction / Feasibility / Tradeoff. Output: concrete recommendation + conditions ("Pursue X if you can own design and rollout, and agree with your manager which existing work moves"). Qualitative, not another numerical score. Not useful for ~3-6 months on new jobs where project choice is limited anyway. Source: ChatGPT analysis item #4.

- **[deferred] Review / promotion brief generation.** New export command producing a performance or promotion-conversation document: 3-4 contribution themes + strongest evidence + user's contribution (vs. collaborators) + connection to current + next-level expectations + remaining gaps + a discussion request. For promotion specifically: confirmed support (who said what, with specificity) and decision process (who participates, what evidence they want, timing constraints). Reuses existing Artifact Library + Feedback Stream + Measures Tracker. Not needed for 6+ months. Source: ChatGPT analysis item #5.

## Polish & observations — from dogfood

- **[polish] `/vibe-check` — gate trust-column display on Manager Background Name being non-empty.** Currently the trust columns display `Medium / Medium` on a Manager row that has no Name (right after schema migration). Technically correct, but weird UX. Hide trust until there's a Name on file.
- **[polish] `/vibe-check` — add "no interactions yet — vibe TBD" interpretation line** for freshly-ingested people with zero Trust Log events. Currently vibe-check.md's "Interpretation lines" section only has examples for populated cases.
- **[polish] `/kickoff` — codify "context-aware option menus" as a reusable prompting pattern.** During `/kickoff` dogfood, Step 6 Q2 ("biggest career concern") worked well because I inlined options drawn from session context (e.g., "the IC-vs-management branch point" because the user had just named it). That pattern should be documented in a reference so other free-text-field prompts can adopt it instead of asking blind.

## Open design questions

- **Does Critical feedback from `MGR` / `K###` via `/log-feedback` auto-log a Trust Log entry?** Current design: `/log-feedback` writes a Feedback Stream row and (for Critical from `MGR` / `K###`) drops a Coaching Note; it does NOT auto-touch Trust Log. The ChatGPT analysis around commitments reinforces that trust capture should stay explicit (opt-in via `/vibe-check`). Leaving as-is for now, but revisit if dogfood shows users forget to run `/vibe-check` after Critical feedback.
- **Should `/ingest-manager` and `/ingest-stakeholder` capture an initial Trust assessment, or stay at the Medium default?** Current design: Medium default; signal moves them. Alternative: ask the user at ingest time ("working relationship so far: solid / unclear / rough"). Adds friction; may or may not pay off. Likely stay default.
- **Mode detection cue list is growing (currently 24 numbered rules).** Consider priority tiers (hot cues first) or categorization (ingest / capture / review / relationships / session) if list hits ~30.
- **Does project-selection warrant a `career-strategy` top-level section in `coaching_state.md`?** Not required for just one project-comparison workflow, but if more career-strategy capabilities land (negotiation prep, scope-expansion prep, etc.), a dedicated section may make sense.

## Working principles for this file

- Public roadmap, no personal data (that lives in `coaching_state.md`, gitignored).
- Items move: `[next up]` → `[in progress]` → removed on merge (git log is the authoritative done record).
- `[deferred]` items keep their rationale so a future contributor (or future me) knows why they were deferred, not just that they were.
- Observations from dogfood are first-class — they're often the most valuable items because they're grounded in actual use.
- Don't let this file become a spec dumping ground. Each item should be one short paragraph. Full designs go in `references/` when the item is actually built.
