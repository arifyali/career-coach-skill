# PDF Ingestion Protocol

Applies to `/ingest-manager`, `/ingest-stakeholder`, and `/ingest-leveling`. The user supplies a PDF path (LinkedIn "Save to PDF" export, team leveling doc, OKR exports). Claude Code reads PDFs via the Read tool natively — no scraping, no external services.

## General flow

1. **Confirm path.** User provides an absolute PDF path. If relative or ambiguous, ask for the absolute path. If the path is not a `.pdf`, confirm the user intends a non-PDF file and fall back to "paste the text instead."
2. **Read.** Use the Read tool directly on the path. For PDFs under 10 pages, read without `pages`. For larger PDFs (common for leveling docs), request a page range first — ask the user which pages are the relevant ones, or scan a short early range to find the index.
3. **Extract.** Follow the type-specific schema below. Produce a "What I Extracted" summary with per-field confidence labels (`High / Medium / Low`).
4. **Gap questions.** Generate targeted follow-ups **one at a time**, prioritized by coaching relevance. Confirm or correct inferred fields; fill in blanks the PDF doesn't cover.
5. **Write to state** with `Last enriched: [date]` and the confidence label. User-confirmed fields are `High`; PDF-inferred fields are `Medium`; the user's recall-only answers are `Medium` or `Low` depending on how specific they are.

## Type: LinkedIn (manager or stakeholder)

### Extraction schema

Pull these from the PDF:

- **Name**
- **Headline** (the subtitle under their name — often reveals how they publicly frame themselves)
- **Current title + company + start date** (anchors tenure)
- **Experience timeline**: title, company, dates, one-line scope/responsibility per role. Walk the full list. Note transitions (eng->data, IC->mgmt, startup->enterprise).
- **Education**: degree, school, dates. Flag anything unusual (non-traditional path, late-career degree).
- **Skills** (the LinkedIn skills section, if included)
- **Certifications / Publications / Patents**
- **About / Bio paragraph** — critical; this is their *self-chosen* framing

### Gap questions (prioritized)

Ask in this order, one at a time. Stop early if the user says "that's enough" or "I don't know yet."

For a **manager**:

1. "The LinkedIn shows they moved from [X role] to [Y role] around [date] — do you know why? What did they tell you about that transition?" (motivation signal)
2. "What do they talk about as their biggest wins? Not from LinkedIn — from your actual conversations." (what they're proud of = what they measure others on)
3. "What do they measure you on that's **not** on the leveling doc?" (the invisible scorecard)
4. "How do they prefer to communicate — written, Slack, 1:1, async docs? What's their decision style?" (working-style signals)
5. "What are they currently under pressure from — their skip-level, a peer leader, a shipping deadline?" (upstream pressure)
6. "What have they explicitly asked you to focus on in the next quarter?" (explicit priorities)
7. "Where are they likely to push back on you — what topics have been contentious?" (open gaps / known friction)

For a **stakeholder**:

1. "What's your working relationship — collaborator, customer, blocker, peer, sponsor?" (relationship type field)
2. "What do they want from your team this half? What's their team's OKR you're on the hook for?"
3. "How often do you interact — standing meeting, async, ad-hoc?"
4. "What have they praised about your work? What have they pushed back on?"
5. "Is there a decision this stakeholder has recently made that affects how you should be working?"

Not every question needs to fire. Stop when the user signals they want to move on.

### Write to state

**Manager** → populate `Manager Background` section. Keep PDF-inferred items in "Career arc" and "Prior companies / roles". Keep user-confirmed items in "Explicit priorities", "Working-style signals", "Open gaps". Set `Confidence: Medium` initially; a later `/log-1-1` entry that confirms a field upgrades it to `High`.

**Stakeholder** → append a new row to Stakeholder Backgrounds table with an auto-assigned `K###` ID. Create a corresponding `### Stakeholder Details -> #### K### — [Name]` block. If the user says this stakeholder already exists, update the existing block instead of creating a new one — ask for the `K###` or stakeholder name explicitly before writing.

## Type: Leveling doc

### Extraction schema

Leveling docs vary widely (Google-style ladders, O3 ladders, bulleted competencies, scored rubrics). Extract whatever structure the doc provides:

- **Role title and level labels** (e.g., L3 / L4 / L5, or IC2 / IC3 / Senior / Staff)
- **The user's current and target levels** (ask the user which rows apply)
- **Competency areas** — the top-level buckets the doc groups by (e.g., Technical, Business Impact, Leadership, Communication)
- **Per-level criteria** within each competency — the bullets or paragraphs

### Generating measures

For each criterion that applies to the user's target level, draft a Performance Measure row:

- `Measure`: a tight restatement of the criterion in action-verb form (not a direct copy — if the doc says "Demonstrates strong technical judgment", the measure is "Demonstrates technical judgment in design reviews and architectural decisions"). Compress to 1-2 lines.
- `Source`: the doc section + heading path (e.g., `Signifyd DS Ladder -> L4 -> Technical Impact`). Enough that the user can find the original.
- `Current Maturity`: start at `Not Yet Demonstrated`. Will be updated by `/review-measures`.
- `Target Maturity`: `Meeting` by default (the user needs to be at Meeting+ on L4 criteria to be promoted to L4). If the user explicitly wants to exceed the target, set `Exceeding` for selected measures.
- `Trend`: `Insufficient Signal` (no history yet).
- `Priority Weight`: computed by cross-referencing against `Company Context.Role-to-priority mapping`. See the Priority Weight section in `rubric-performance.md`. On first ingest, mark each measure High/Med/Low per that logic. If `Company Context` is empty, mark all as `Med` and leave a Coaching Note: "Priority Weights unset — run /log-company-goals or re-run kickoff to tune."
- `Supporting Artifact IDs`: empty.
- `Last Reviewed`: empty.

### Gap questions (prioritized)

Only ask after extracting the raw measures. Fire these when relevant:

1. "Measure [M###] — this one reads vaguely as '[criterion]'. What does your manager say that looks like concretely?" (fire for each measure where the extracted criterion is ambiguous)
2. "Are there competency areas you expected to see in this doc but didn't?" (surface informal expectations)
3. "Is this doc authoritative, or is there a side-channel for how your manager actually calibrates? (e.g., peer feedback, 360s, performance committee)" (reveal shadow rubrics)
4. "Which 2-3 measures are the ones your manager has already flagged as most important for your next review?" (user-identified priority, overrides automatic Priority Weight)

### Write to state

Append rows to Performance Measures Tracker. Compose IDs as `M###`, starting from `M001` on first ingest and continuing from the highest existing ID on re-ingest. Set `Role profile source` in Profile if the leveling doc implies a different role than previously recorded; ask the user to confirm.

## Re-ingestion

Users will re-ingest a leveling doc when the company updates it, or a new LinkedIn PDF when a manager/stakeholder changes roles. On re-ingest:

- **Diff first.** Show "What's new / changed / removed" against the existing state before writing.
- **Preserve history.** Never silently delete rows or Details blocks. If a measure disappears from the new leveling doc, append `(deprecated [date])` to its Measure field but keep its Supporting Artifact IDs for provenance.
- **Confirm, then write.** The user must say OK before anything persists.

## Privacy note

Supplied PDFs are read, not stored. The skill extracts fields and writes those into `coaching_state.md` (gitignored). PDFs the user wants to retain can live in the gitignored `uploads/` directory, but the skill does not require nor automate storage. The default is: the PDF stays wherever the user has it (Downloads folder, etc.) and nothing is copied into the repo.
