# people — Look up manager and stakeholder IDs

Lists the manager (`MGR`) and every stakeholder (`K###`) on file with the fields needed to pick the right FK when running `/log-feedback`, `/prep-stakeholder`, `/log-stakeholder-meeting`, or `/log-1-1 --with`. Read-only — does not modify state.

## Invocation

```
/people                   # list everyone
/people priya             # filter by case-insensitive name substring
/people --stale           # only rows where Last Interaction > 60 days (chase stale)
/people --role Reviewer   # filter by Relationship Type (stakeholders only)
/people --trust-flags     # only rows with any Low trust (either direction) or an active trust flag
```

Filters compose: `/people --stale --role Reviewer`.

For deep trust state on one person (full Trust Log, event logging), use `/vibe-check <id-or-name>` — see `references/commands/vibe-check.md`.

## Flow

### Step 1 — Read state

Load `coaching_state.md`. Pull Manager Background + the Stakeholder Backgrounds table. No writes.

### Step 2 — Apply filters

- Positional substring: case-insensitive match against Name on both Manager Background and Stakeholder Backgrounds rows.
- `--stale`: keep rows where `Last Interaction` is empty or more than 60 days ago. Manager Background uses `Last enriched` as the staleness proxy.
- `--role <Type>`: canonical Relationship Type (from `references/stakeholder-relationship-types.md`). Multi-type rows match if the type is present. Does not apply to Manager Background — manager is always included unless the substring filter excludes them.
- `--trust-flags`: keep rows where either Trust column is `Low`, OR the person has any trust flag defined in `references/commands/vibe-check.md` roster-mode flag rules (negative events in 30d, missed commitments in 60d, decay pending, low-trust signal source on a High-weight measure).

### Step 3 — Render

Compact table with the FK front and center. Manager row first, then stakeholders sorted by Relationship Type (Senior Partner / Reviewer first, then alphabetical).

```markdown
## People on file — [N] shown ([total] total)

| ID | Name | Role | Team | Relationship Type | Last Interaction | Trust (them→you) | Trust (you→them) |
|---|---|---|---|---|---|---|---|
| MGR | [manager name] | [title] | [team] | manager | [date or "—"] | Med | High |
| K001 | [name] | [role] | [team] | Partner | [date] | High | Med |
| K002 | [name] | [role] | [team] | Reviewer | 2026-08-14 (67 days) | Low | Med |
```

If no matches:
> No matches for filter `[filter]`. Known people: [count]. Try `/people` with no filter, or `/ingest-stakeholder <pdf>` to add someone new.

If Stakeholder Backgrounds is empty entirely:
> No stakeholders on file yet. **Recommended next**: `/ingest-stakeholder <pdf>` to add one. The manager row above is from `/ingest-manager`.

### Step 4 — Prescriptive next-move nudges

Only when the output clearly implies action — keep it one line:

- **Stale rows present (not using `--stale`)**: "N stakeholders haven't been touched in 60+ days. `/people --stale` to see them."
- **Low-trust rows present (not using `--trust-flags`)**: "N people have Low trust (them→you or you→them). `/people --trust-flags` for the list; `/vibe-check <id>` for the Trust Log."
- **No manager on file**: "Manager Background is empty. `/ingest-manager <pdf>` adds the FK that `/log-1-1` and `/prep-1-1` resolve against."
- **Freshly-created `Peer (<original>)` rows from schema migration**: "N stakeholders have placeholder Relationship Types from the v1 → v2 enum migration. Walk them with `/ingest-stakeholder` or edit by hand."

### Step 5 — Session Log (optional)

Append only if the user ran `/people` explicitly as a command (not as a resolution helper mid-command): `people | filter=[filter], [N] shown`. Keep the Session Log tight — skip if no filter and no action implied.

## When to use

- You want to attribute feedback in `/log-feedback` but can't remember the stakeholder's `K###`.
- You're drafting a `/prep-stakeholder` roster and want to pick 2-3 people without opening `coaching_state.md`.
- You want to see which stakeholders have gone cold (haven't been in a meeting or feedback entry recently).
- You're spot-checking the Relationship Type field after `/ingest-stakeholder` ran — the row should match the canonical enum in `references/stakeholder-relationship-types.md`.

## Output Schema

```markdown
## People on file — [N] shown ([total] total)

| ID | Name | Role | Team | Relationship Type | Last Interaction | Trust (them→you) | Trust (you→them) |
|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... |

[optional one-line nudge from Step 4]
```

No `Recommended next` block unless Step 4 fired — `/people` is a lookup, not a workflow step.
