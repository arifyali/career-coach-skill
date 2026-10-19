# export-artifacts — Hand off to interview-coach-skill

Read `references/interop-interview-coach.md` for the full interop spec. This file is the command wrapper.

## Flow

### Step 1 — Resolve the export path

- Read `Profile.Interview-coach export path`.
- If the value is `none`: respond "No export path set. Set one via `/kickoff` or run `/export-artifacts <absolute-path>` to export once."
- If the value is relative (e.g., `../interview-coach-skill/inbox/career-artifacts.md`): resolve to absolute, treating the skill root as CWD.
- If the parent directory doesn't exist AND the path is NOT the default `inbox/` convention: abort and ask the user to confirm. Rationale: avoid silently creating unexpected directories.

### Step 2 — Apply optional filters

- `/export-artifacts since 2026-06-01` — only artifacts dated on/after.
- `/export-artifacts measure M003` — only artifacts linked to the specified measure.
- `/export-artifacts last-review` — only artifacts dated after the most recent Progress Snapshot.
- `/export-artifacts include-low-confidence` — include Low-confidence artifacts (default excludes them).

If no filter: export all High and Medium confidence artifacts.

### Step 3 — Build the file

Format per `interop-interview-coach.md`. Required elements:

- Leading header: `# Career Artifacts — exported [date]`
- Source line pointing at the state file
- Artifact count line
- One block per artifact with X, Y, Z, Linked measures, Competency tags, Confidence, Evidence sources, Date

### Step 4 — Write

- Overwrite the file at the resolved path (idempotent).
- Create the parent directory ONLY if it is `inbox/` under `interview-coach-skill`. Otherwise abort per Step 1.
- Append Session Log row: `export-artifacts | exported [N] artifacts to [path] [filter note if applied]`.

### Step 5 — Confirm and suggest handoff

"Exported [N] artifacts to [path]. In interview-coach-skill, run `/stories add --from-career-coach` to seed STAR drafts."

## Edge cases

- **Zero artifacts match the filter**: don't write the file. Tell the user: "No artifacts match that filter. Nothing exported."
- **All artifacts are Low confidence**: warn before writing. "All [N] matching artifacts are Low confidence. Export anyway, or filter?"
- **Interview-coach-skill not installed**: the default path won't resolve. Tell the user and offer alternatives: "Default path doesn't exist. Options: install interview-coach-skill, set `Profile.Interview-coach export path` to `none`, or export to a different path."

## Output Schema

```markdown
## Artifacts Exported — [date]
- Path: [absolute path]
- Count: [N]
- Filter: [none / since / measure / last-review / include-low-confidence]

### What Was Exported
- [list of A### — titles]

**Recommended next**: In interview-coach-skill, run `/stories add --from-career-coach`. **Alternatives**: `/review-progress`, `/log-artifact`
```
