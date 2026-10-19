# export-state — Bundle state for machine moves / backups

Writes a snapshot of `coaching_state.md` to a user-supplied path. Enables moving between machines or taking a point-in-time backup. This is the portability command — mentees get their state along for the ride without touching the public skill repo.

## Flow

### Step 1 — Determine target path

- Explicit: `/export-state <path>` → use that absolute path.
- No arg: default to `~/career-coach-backup-YYYYMMDD.md` where YYYYMMDD is today's date. Expand `~`.
- Confirm with the user: "I'll write a state bundle to [path]. OK?"

### Step 2 — Compose the bundle

A state bundle is a single markdown file containing:

1. Header: `# Career Coach State Export — [date]` + source path + bundle format version.
2. The full current `coaching_state.md` content inlined under `## coaching_state`.
3. Optional appendices:
   - If the user has supplementary notes they want to persist (e.g., a long-form self-review draft), ask whether to include. By default, bundle is state-only.
4. Trailing checksum line: `# Bundle sha256: <hash of coaching_state content>` — lets `/import-state` detect corruption.

### Step 3 — Write

- Write the bundle to the target path.
- Do NOT modify `coaching_state.md`.
- Append Session Log row: `export-state | bundled to [path]`.

### Step 4 — Confirm and suggest next steps

"Bundle written to [path]. To restore on another machine: clone the career-coach-skill repo, copy the bundle to the new machine, run `/import-state <path>`."

## Safety

- Never write to any location inside the skill repo root — the bundle contains personal data and must not accidentally be committed. If the user supplies a path inside the repo, abort: "That path is inside the skill repo. Pick a path outside — your home directory is safe."
- Warn about privacy: "This bundle contains your 1:1 notes, feedback, and artifacts. Store it like you would any sensitive file."

## Output Schema

```markdown
## State Exported — [date]
- Path: [absolute path]
- Size: [N lines of state content]
- Checksum: [sha256 fragment]

### Restore Instructions
On the target machine:
1. Clone career-coach-skill from GitHub
2. Copy the bundle to the new machine (or sync via your preferred mechanism)
3. Run `/import-state <path>` in the new skill directory

**Recommended next**: continue your current session, or run `/status` for a trajectory read. **Alternatives**: `/log-artifact`, `/review-progress`
```
