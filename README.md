# career-coach-skill

A persistent, evidence-driven career coach for data professionals — built as a Claude Code skill. Tracks performance against your company's leveling criteria, enriches your model of your manager and stakeholders over time, logs 1:1s and inbound feedback, builds a library of accomplishments in Google XYZ format, and shows you how you're trending toward your next career stage.

Designed to flex across Data Scientist, Analytics Engineer, BI Engineer, Data PM, and generic data roles. Hands off to [`interview-coach-skill`](https://github.com/noamseg/interview-coach-skill) when it's time to prep interviews.

## What it does

- **Tracks performance against your actual leveling rubric.** Ingest your company's leveling doc as a PDF (or paste it in); the skill extracts each criterion as a trackable measure.
- **Enriches your model of manager and stakeholders.** Export their LinkedIn as PDF; the skill parses it and asks targeted gap questions ("what are they measuring you on that's *not* on the leveling doc?").
- **Captures 1:1s and feedback as first-class inputs.** A 1:1 entry can update manager background, add feedback to the stream, and refresh the review date on linked measures — all from one flow.
- **Captures accomplishments in Google XYZ format.** "Accomplished X as measured by Y, by doing Z." The quantified-Y rule is enforced — vague wins don't make it into the library.
- **Shows you how you're trending.** `/review-progress` computes a Momentum Score (0-100) and a Trajectory Call (On Track / Lagging / At Risk / Ahead) relative to your short-term career goal. The formula is simple and visible — you can dispute specific inputs.
- **Hands off to interview-coach-skill.** `/export-artifacts` emits your XYZ library in a format `interview-coach-skill` can seed STAR drafts from. Career coaching and interview coaching stay separate but connected.

## Commands at a glance

Day to day you mostly need the rhythms: `/daily`, `/weekly`, and `/monthly`. They show what's on deck, ask a few guiding questions, and route your answers into the capture commands below so you don't have to pick one yourself. Weekly and monthly debriefs start from a recap drafted from the level below, which you correct and extend. Everything else is there for when you want to go direct.

| Phase | Command | When to use |
|---|---|---|
| Rhythm | `/daily [pre\|debrief]` | Start and end of each day — quick brief, guided prompts, commitment flips |
| Rhythm | `/weekly [pre\|debrief]` | Start and end of each week — recap from your dailies, `/review-progress`, bulk commitment flips, trust review |
| Rhythm | `/monthly [pre\|debrief]` | Start and end of each month — set a focus; recap from your weeklies, scoped `/review-measures` |
| Setup | `/kickoff` | First run; also when role, level, or career goals change |
| Setup | `/ingest-manager <pdf>` | After kickoff; whenever manager changes |
| Setup | `/ingest-stakeholder <pdf>` | Per key stakeholder; repeat as team shifts |
| Setup | `/ingest-leveling <pdf-or-paste>` | After kickoff; re-run when the leveling doc updates |
| Meetings — prep | `/prep-1-1 [--with <ids>] [artifact]` | Before a 1:1 with your manager (optionally joint) |
| Meetings — prep | `/prep-stakeholder <id1> [id2 ...] [--with-manager] [--with-skip] [artifact]` | Before a stakeholder meeting; 1+ participants |
| Capture | `/log-artifact` | Shipped something measurable |
| Capture | `/log-1-1` | After a 1:1 (auto-debriefs against matching prep) |
| Capture | `/log-stakeholder-meeting <id-or-name>` | After a stakeholder meeting (prep-aware) |
| Capture | `/log-feedback` | Standalone feedback (Slack, email, review doc) |
| Capture | `/commitment [add\|<C###>\|--from-paste\|--mine\|--waiting\|--to\|--overdue]` | Track accepted commitments; auto-proposes Trust Log entries on status flips |
| Review | `/review-measures` | Runs inside `/monthly debrief`; directly when evidence feels thin on a measure |
| Review | `/review-progress` | Runs inside `/weekly debrief`; definitely before any formal review cycle |
| Review | `/log-company-goals` | New quarter, OKR refresh, team priority shift |
| Portability | `/export-artifacts` | Starting interview prep |
| Portability | `/export-state` | Backing up or moving machines |
| Portability | `/import-state <path>` | Restoring state on a new machine |
| Relationships | `/people [name] [--stale] [--role <Type>] [--trust-flags]` | Look up manager + stakeholder IDs; find who's gone cold or low-trust |
| Relationships | `/vibe-check [id-or-name]` | Trust tracking — roster scan or deep view + log for one person |
| Session | `/status` | Session start (auto); or explicitly for a trajectory read |
| Session | `/help` | Reminder of this list in-session |

## Quick start

### 1. Clone this repo

```sh
git clone https://github.com/<your-fork>/career-coach-skill.git
cd career-coach-skill
```

### 2. Activate the skill

Claude Code picks up skills from the current working directory. From this repo's root, open a Claude Code session and run `/kickoff`.

The first time you run `/kickoff`, the skill:

- Copies `coaching_state.example.md` to `coaching_state.md` (gitignored — stays local).
- Asks for your role, current level, target level, company, review cycle, short-term career goal, optional long-term goal, company/team priorities, feedback directness, and the path to export artifacts for `interview-coach-skill` if you use it.

### 3. Seed context

After `/kickoff`, the recommended sequence:

```
/ingest-manager <path-to-manager-linkedin.pdf>
/ingest-leveling <path-to-leveling-doc.pdf>
/ingest-stakeholder <path> <role>    # repeat per key stakeholder
```

LinkedIn PDFs: on LinkedIn, go to a profile -> "More" -> "Save to PDF". The skill reads the PDF directly, extracts career arc + prior roles + current scope, then asks targeted gap questions.

### 4. Run the rhythms

```
/daily               # morning: today's meetings + commitments due, then intention / risk / who to raise what with
/daily               # evening: what shipped / unresolved / surprises, then flip commitment statuses
/weekly              # Monday: week ahead, commitments to lock, meetings worth prepping
/weekly              # Friday: recap drafted from your dailies, progress read, commitment flips, trust review
/monthly             # 1st week: pick the month's focus measure and the evidence you want by month-end
/monthly             # last week: recap drafted from your weeklies, measure review, what to change
```

Mode (pre vs. debrief) is inferred from time of day / day of week / day of month; pass `pre` or `debrief` to force it. Answers that sound like a shipped outcome, feedback, or a commitment get offered as a capture — nothing is written until you accept. Skip any prompt; the rituals don't nag.

### 4a. Or capture directly

```
/log-artifact        # log an accomplishment (quantified Y enforced)
/log-1-1             # capture a 1:1 (prep-aware)
/log-feedback        # standalone feedback (Slack, email, review doc)
```

### 4b. Meeting prep + debrief loop

Walk into meetings with an agenda and anticipated questions shaped by your actual state (manager/stakeholder priorities, open gaps, measures with weak evidence, pending feedback). Walk out and debrief against the prep so drift gets caught.

```
# Prep
/prep-1-1                                       # pure 1:1 with your manager
/prep-1-1 --with K002 ./review-draft.pdf        # joint 1:1 (manager + one stakeholder, with artifact)
/prep-stakeholder K002                          # single stakeholder meeting
/prep-stakeholder K002 K007 --with-manager      # roadmap sync with manager + two stakeholders

# Debrief
/log-1-1                                        # auto-detects a matching Meeting Prep (manager in roster)
/log-stakeholder-meeting K002                   # auto-detects a matching Meeting Prep (stakeholder in roster)
```

Each prep writes a Meeting Prep row with a copy-paste-ready agenda block for your calendar invite, plus per-participant anticipated questions and user-side questions-to-ask (grouped by person, so you know who to ask what). Debrief flows walk the agenda against reality and run per-participant drift checks against each person's profile — Relationship Type shifts, priority changes, and closed/opened gaps all surface and offer background updates.

Multi-participant meetings are supported end-to-end: joint 1:1s, roadmap reviews, cross-team syncs. Either debrief command (`/log-1-1` or `/log-stakeholder-meeting`) matches any prep where its target is in the participant list, so joint meetings can be debriefed via whichever entry point feels natural.

### 5. Check trajectory

```
/review-measures     # per-measure coverage + prescriptive next move
/review-progress     # Momentum + Trajectory Call + Progress Snapshot
```

Review monthly; definitely before any formal review cycle.

### 6. Export for interview prep

When it's time to prep interviews, `/export-artifacts` writes your XYZ library to `../interview-coach-skill/inbox/career-artifacts.md` by default (configurable at `/kickoff` or per-run). Clone [interview-coach-skill](https://github.com/noamseg/interview-coach-skill) next to this repo so that default path resolves (`git clone https://github.com/noamseg/interview-coach-skill.git ../interview-coach-skill`). Then in interview-coach-skill, run `stories add` and point it at that file to seed STAR drafts from your XYZ artifacts. (A dedicated `--from-career-coach` import is proposed in `references/interop-interview-coach.md` but isn't in interview-coach-skill yet.)

## Public vs. private — what's in the repo

**Committed (public, shareable):**

- `SKILL.md`, `README.md`, `LICENSE`, `.gitignore`
- Everything under `references/` and `references/commands/`
- `coaching_state.example.md` (empty schema template)

**Gitignored (private, stays local):**

- `coaching_state.md` — your actual state
- `uploads/`, `tmp/`, `*.pdf` — ingested PDFs
- `*.local.md` — scratch
- `career-coach-backup-*.md` — state exports

The split means you can share this repo publicly (GitHub, with mentees, with coworkers) without leaking your 1:1 notes, feedback, or accomplishments.

## Moving between machines

```
/export-state                              # writes to ~/career-coach-backup-YYYYMMDD.md
# (copy the bundle to the other machine)
/import-state ~/career-coach-backup-20270115.md
```

Bundles are plain markdown with a checksum. The `/import-state` flow requires explicit confirmation before overwriting any existing state.

## Deploying to mentees

The skill is written to flex across data roles via `references/role-profiles.md`. For each mentee:

1. They clone this repo (your fork or main — either works).
2. They run `/kickoff` fresh — generates their own `coaching_state.md`. Nothing in your state is shared.
3. They pick their role profile at kickoff; the skill adapts example phrasings and seed measures to that role.
4. If they don't use `interview-coach-skill`, they set `Profile.Interview-coach export path` to `none` and the handoff layer is dormant.

Supported role profiles: `data-scientist`, `analytics-engineer`, `bi-engineer`, `data-pm`, `generic-data`.

## Design philosophy

- **Evidence over narrative.** Every maturity call, Momentum Score, and Trajectory Call cites artifact IDs, feedback dates, or Meeting Log rows. If evidence is weak, the skill says so.
- **Friction where it matters.** The quantified-Y guardrail on artifacts, the explicit confirmation on state overwrites, the user-confirmed updates on Manager Background — these are the parts that prevent a library of plausible-sounding nothing.
- **Trust is separate from performance.** `/vibe-check` tracks signal reliability (them → you) and your follow-through (you → them) per person. It surfaces career-progression risks (low-trust Reviewer, missed-commitment patterns) but does **not** affect Momentum Score. Relationship signal and performance signal stay in different lanes, both visible.
- **Commitments are the concrete practice behind the relationship goal.** `/commitment` captures accepted promises with a Status lifecycle (Open → Blocked → Completed / Renegotiated / Cancelled). Status flips to Completed or Missed auto-propose Trust Log entries on the Owed-to person, with confirmation. Only commitments the user explicitly accepted land in the table — suggestions don't become promises until the user says so.
- **Simple scoring, visible math.** The Momentum formula is a short weighted average. The response shows every input. You can dispute specific maturity calls, not an opaque score.
- **One source of truth per concept.** XYZ artifacts live here. STAR enrichment lives in `interview-coach-skill`. Performance measures live here. Interview stories live there. The handoff is one-way and file-based.
- **State is yours.** Nothing phones home. Everything persists to a single gitignored file you own.

## Command reference

See **Commands at a glance** near the top of this README for the full table. In-session, run `/help` for the same list; the canonical source is `references/commands/help.md`.

## License

MIT. See `LICENSE`.
