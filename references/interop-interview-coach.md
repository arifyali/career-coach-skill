# Interop with interview-coach-skill

One-way, file-based export. Career-coach-skill writes an artifacts file; interview-coach-skill reads it when the user runs `/stories add --from-career-coach`. There is **no reverse sync** and **no shared state file**.

## Why a file export, not shared state

- The two skills have different schemas and update cadences. Coupling them forces dual migrations on every schema change.
- Many users of career-coach-skill will not use interview-coach-skill (and vice versa). Mentees in particular may use only one.
- File export is reproducible, inspectable, and the user can hand-edit it before interview prep.

## Export path

Stored in `Profile.Interview-coach export path`. Set at `/kickoff`. Options:

- **Default**: `../interview-coach-skill/inbox/career-artifacts.md` (relative to the career-coach-skill directory). Resolve to an absolute path at export time.
- **Absolute path**: any user-specified absolute path.
- **`none`**: `/export-artifacts` becomes a no-op (returns a message like "No export path set. Run `/kickoff` and set the path, or run `/export-artifacts <path>` to export once.").

The `inbox/` directory under `interview-coach-skill` is a well-known drop location; create it if missing (and only there — never create unexpected directories elsewhere).

## Export format

One block per artifact, plain markdown, no YAML frontmatter. The file is overwritten in full on each export (idempotent — the career-coach Artifact Library is the source of truth; the export is a snapshot).

```markdown
# Career Artifacts — exported [date]
Source: career-coach-skill — <absolute path to coaching_state.md>
Artifact count: N

---

## A001 — [Title]
- Date: [date]
- X: [Accomplished …]
- Y: [Measured by …]  (or `non-quant-reason: …`)
- Z: [By doing …]
- Linked measures: [M001, M003]
- Competency tags: [list]
- Confidence: [High / Medium / Low]
- Evidence sources: [list]

---

## A002 — [Title]
...
```

The leading `# Career Artifacts — exported [date]` header is required — it's how interview-coach-skill confirms the file is a career-coach export vs. an unrelated markdown file.

## `/export-artifacts` behavior

- Reads full Artifact Library + Artifact Details from `coaching_state.md`.
- Optional filter arguments:
  - `/export-artifacts since 2026-06-01` — only artifacts dated on/after.
  - `/export-artifacts measure M003` — only artifacts linked to M003.
  - `/export-artifacts last-review` — only artifacts dated after the most recent Progress Snapshot.
- Writes the full file at the configured path. Overwrites any existing file at that path (idempotent).
- Appends a Session Log row: `Exported N artifacts to <path>`.
- If the path's parent directory doesn't exist and is NOT `../interview-coach-skill/inbox/`, abort and ask the user to confirm the path or create the directory manually. Rationale: avoid silently creating unexpected directories.

## interview-coach-skill read path

interview-coach-skill's `references/commands/stories.md` gains a short block describing `/stories add --from-career-coach`:

1. Read the career-artifacts file at the configured path (ask the user if not configured).
2. Verify the leading header matches `# Career Artifacts — exported`.
3. Present a numbered menu of the XYZ blocks.
4. For each selected block, create a draft Story record in the Storybank with:
   - **Title**: the artifact Title (user can rename).
   - **Situation** seed: inferred — ask the user to fill in from the Z description and their memory.
   - **Task** seed: inferred — ask the user.
   - **Action** seed: Z, lightly rephrased.
   - **Result** seed: Y, lightly rephrased.
   - **Earned Secret**: empty — must be extracted via the normal `references/differentiation.md` protocol. STAR scaffolding is seeded, but the differentiation layer stays in interview-coach-skill.
   - **Primary Skill / Secondary Skill**: derived from the artifact's Competency Tags; user confirms.
   - **Strength**: start at 2 (needs improvement — it's a draft).

The user then runs `/stories improve S###` to upgrade the draft to a 4+ interview-ready story. The hand-off is: career-coach gives you the raw ingredients and quantified Y; interview-coach shapes it into a story with narrative tension and Earned Secret.

## What NOT to export

- Manager / stakeholder names and background details — none of this is in XYZ format and none of it belongs in Storybank entries.
- 1:1 Log and Feedback Stream rows — operational notes, not resume material.
- Company Context — relevant for performance tracking, not for interviews outside the current company.
- Low-confidence artifacts, by default (users can override with `/export-artifacts include-low-confidence`). Rationale: a bad artifact becomes a bad story.

## Mentee deployment

Mentees cloning this skill from GitHub inherit the default export path pointing to a sibling `interview-coach-skill/` directory. If they don't have interview-coach-skill installed, `/kickoff` lets them set the path to `none` and the interop layer is dormant. They can set a real path later via `/kickoff` re-run or by hand-editing `Profile.Interview-coach export path`.
