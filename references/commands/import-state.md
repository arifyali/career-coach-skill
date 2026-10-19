# import-state — Restore from a bundle created by /export-state

Reverse of `/export-state`. Overwrites `coaching_state.md` with the bundle contents. **Requires explicit user confirmation** before overwriting any existing state.

## Flow

### Step 1 — Validate the bundle

- `/import-state <path>` — read the path.
- Confirm the file exists and starts with `# Career Coach State Export —`. If not, abort: "Doesn't look like a career-coach bundle. First line was: [first line]."
- Verify the checksum if present. If mismatch, warn: "Checksum doesn't match the bundled coaching_state content. The bundle may be corrupted. Proceed anyway? (yes/no)"

### Step 2 — Preview

Show a one-screen diff summary:

- Current state (if any): "N artifacts, M measures, K 1:1 Log rows, last session [date]."
- Bundle state: "N' artifacts, M' measures, K' 1:1 Log rows, last session [date]."
- Call out losses: "Importing will overwrite your current state. [X difference] will be lost unless you also run `/export-state` first."

### Step 3 — Confirm

"Overwrite current coaching_state.md with the bundle? (yes/no)"

Only `yes` proceeds. Any other response aborts cleanly.

### Step 4 — Write

- Replace `coaching_state.md` with the bundled content (extracted from the `## coaching_state` section of the bundle).
- Append a Session Log row to the newly-restored state: `import-state | restored from [path] (previous state had [N] artifacts, [M] measures)`.
- Update the `Last updated:` line at the top of `coaching_state.md` to today's date.

### Step 5 — Confirm success

"State restored from [path]. Run `/status` to get a fresh trajectory read."

## Edge cases

- **No existing `coaching_state.md`**: skip the diff + loss warning; proceed directly to confirmation.
- **Current state is unmodified from `coaching_state.example.md`**: safe to import without warning (nothing real will be lost).
- **Bundle version mismatch**: if the bundle format version is newer than this skill knows about, warn but allow import. Run the Schema Migration Check (`references/schema-migration.md`) after the restore.

## Safety

- Never silently overwrite. Confirmation is non-negotiable.
- If the user is importing on a machine that may be shared, remind them: "Make sure this is your own machine or dev container — the state file contains personal 1:1 notes, feedback, and artifacts."

## Output Schema

```markdown
## State Imported — [date]
- Source bundle: [path]
- Content restored: [N artifacts, M measures, K 1:1 rows]
- Prior state: [overwritten / none to overwrite]

**Recommended next**: `/status` for a trajectory read with the imported state. **Alternatives**: `/review-progress`, `/log-artifact`
```
