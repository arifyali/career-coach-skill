# Role Profiles

Role-flex config. `/kickoff` sets `Role profile source` in Profile to one of the keys below. Every downstream command reads that pointer and uses the role's defaults for competency tags, seed measures (when no leveling doc is ingested yet), and example XYZ phrasings.

Rubric is role-agnostic (see `rubric-performance.md`); the MEASURES come from the user's actual leveling doc via `/ingest-leveling`. Role profile only **seeds defaults** when no doc exists yet, and shapes **prompt phrasing** across all commands.

Keys: `data-scientist`, `analytics-engineer`, `bi-engineer`, `data-pm`, `generic-data`.

---

## data-scientist

**Default competency tags**: Modeling, Experimentation, Causal Inference, Metrics Design, Data Engineering (ETL), Communication, Stakeholder Translation, Code Quality, Business Judgment, Research Rigor.

**Seed measures** (used only if `/ingest-leveling` hasn't been run):
- Scopes and ships end-to-end data science projects (problem framing -> model/analysis -> deploy -> measurement)
- Designs experiments and reads results with appropriate statistical rigor
- Translates ambiguous business questions into tractable data problems
- Communicates findings so non-technical stakeholders act on them
- Partners with engineering to productionize models and pipelines
- Mentors junior data scientists / reviews others' analyses

**Example XYZ phrasings**:
- "Accomplished a reduction in false-positive fraud alerts, as measured by a 18% drop in analyst review volume while holding recall flat, by doing a Bayesian threshold-tuning sweep on the existing XGBoost model and shipping the new cutoffs behind a feature flag."
- "Accomplished faster experiment turnaround for the growth pod, as measured by cutting average experiment analysis time from 6 to 2 days across 12 tests, by doing a reusable notebook template with pre-wired CUPED adjustment and automated p-value / power diagnostics."
- "Accomplished alignment between product and ML on a churn-model refresh, as measured by shipping the new model in Q3 with no scope creep against a Q2 PRD, by doing weekly syncs with explicit tradeoff docs and a shared success metric."

**Top-3 ways typically measured**: project ownership scope, model/analysis quality (reproducibility + rigor), downstream business impact.

---

## analytics-engineer

**Default competency tags**: dbt / Modeling Layers, SQL Depth, Data Quality, Testing & Documentation, Pipeline Reliability, Semantic Layer Design, Stakeholder Enablement, Cost Awareness, CI/CD, Mentorship.

**Seed measures**:
- Owns and maintains production-grade data models (dbt / equivalent) with tests and docs
- Reduces cycle time between raw data landing and stakeholder-usable tables
- Designs semantic / metric layers that stay consistent across BI tools
- Improves pipeline reliability (freshness, uptime, incident rate)
- Enables analysts and PMs via self-serve models and documentation
- Reviews others' SQL / dbt PRs and raises the team's engineering bar

**Example XYZ phrasings**:
- "Accomplished faster dbt builds, as measured by cutting the daily production run from 42 to 11 minutes, by doing an incremental model refactor on the three fact tables that accounted for 70% of compute."
- "Accomplished a trustworthy single-source-of-truth for revenue, as measured by eliminating 100% of the 14 open discrepancies between Looker and the finance dashboard, by doing a semantic-layer consolidation and adding dbt tests on the canonical revenue model."
- "Accomplished self-serve analytics for the growth team, as measured by a 60% drop in ad-hoc SQL requests over two months, by doing documented dbt exposures and a 1-hour onboarding doc per core model."

**Top-3 ways typically measured**: pipeline reliability, model reusability / stakeholder adoption, engineering rigor (tests + docs + reviews).

---

## bi-engineer

**Default competency tags**: Dashboard Design, Metric Definition, Stakeholder Discovery, Looker / Tableau / Power BI Depth, SQL, Information Hierarchy, Narrative Design, Executive Comms, Access Controls, Enablement.

**Seed measures**:
- Delivers dashboards that drive decisions (not just report numbers)
- Turns vague stakeholder questions into clear metric definitions
- Keeps dashboard portfolio healthy (ownership, deprecation, trust)
- Trains stakeholders to self-serve for standard questions
- Partners with analytics engineering on the semantic layer
- Communicates data stories to execs clearly and concisely

**Example XYZ phrasings**:
- "Accomplished executive adoption of the new weekly business review dashboard, as measured by it replacing three legacy decks and being viewed by 100% of the leadership team weekly for 6+ weeks, by doing a discovery pass with each exec and designing the dashboard around the three decisions they actually make."
- "Accomplished a smaller, more trustworthy dashboard footprint, as measured by retiring 22 of 61 dashboards without stakeholder pushback, by doing a usage audit + stakeholder re-confirmation of what they still use."
- "Accomplished faster time-to-insight on marketing campaigns, as measured by cutting campaign-review prep from 2 days to 2 hours, by doing a templated campaign dashboard tied to the central UTM model."

**Top-3 ways typically measured**: decision impact of dashboards, portfolio health, stakeholder enablement.

---

## data-pm

**Default competency tags**: Roadmap Strategy, Stakeholder Alignment, Prioritization, Experimentation Strategy, Data Platform Vision, Executive Comms, Cross-Functional Leadership, Metrics Design, Product Sense, Writing.

**Seed measures**:
- Owns a roadmap tied to measurable team / product outcomes
- Prioritizes between competing requests with explicit tradeoff logic
- Partners with DS / AE / Eng to ship data products end-to-end
- Writes clear strategy docs that leadership can act on
- Drives alignment across stakeholder teams without authority
- Measures and iterates on shipped work

**Example XYZ phrasings**:
- "Accomplished adoption of the new customer-360 data product, as measured by three downstream teams (CX, Marketing, Risk) building production use cases on it within one quarter of launch, by doing a stakeholder-first discovery process and sequencing MVP scope to CX's highest-pain workflow."
- "Accomplished a disciplined experimentation program, as measured by lifting shipped-experiment rate from 2/mo to 7/mo across the growth pod, by doing a weekly intake triage with explicit kill/pursue criteria and a shared template for scoping."
- "Accomplished executive buy-in for the semantic-layer investment, as measured by the roadmap being funded for two engineers in the H2 planning cycle, by doing a one-page business case grounded in support-ticket data and a prototype."

**Top-3 ways typically measured**: outcome of shipped roadmap, stakeholder trust, strategic clarity.

---

## generic-data

Use when the user's role doesn't fit cleanly into the others, OR for a mentee whose role you don't know well yet.

**Default competency tags**: Technical Depth, Business Acumen, Communication, Stakeholder Management, Project Delivery, Code / Analysis Quality, Collaboration, Mentorship, Learning Velocity.

**Seed measures**:
- Delivers data-driven work end-to-end
- Collaborates effectively with technical and business stakeholders
- Produces high-quality, reviewable analytical output
- Grows scope and ownership over time
- Communicates findings so decisions follow
- Mentors / unblocks peers

**Example XYZ phrasings**:
- "Accomplished [outcome], as measured by [quantified change], by doing [specific action/approach]."
- (Encourage the user to drop in two concrete examples from their last six months when running `/log-artifact` for the first time — these replace the generic placeholder.)

**Top-3 ways typically measured**: delivery, impact, growth.

---

## How role profiles are consumed

- `/kickoff` lists the keys and asks the user to pick one. Writes the chosen key into `Profile.Role profile source`.
- `/ingest-leveling` uses the active profile's seed measures as fallback rows if the user has no leveling doc yet (marked with `Source: role-profile seed` so they can be upgraded later).
- `/log-artifact` offers the active profile's competency tags as defaults in step 6 of the capture flow.
- `/review-measures` and `/review-progress` do not change behavior per profile — they're rubric-driven, not role-driven.
- When a user re-runs `/kickoff` with a different role, update `Role profile source`. Preserve Artifact Library and 1:1 Log — only Profile and (optionally, with confirmation) Performance Measures Tracker rows sourced from seeds get overwritten.
