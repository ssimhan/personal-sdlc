---
description: Activate session-scoped safety guardrails that block destructive database, filesystem, and git operations. Use before touching migrations, production data, or force-git operations.
---

# /careful

Activates a safety shield for the rest of this session. Use before running migrations, touching production data, or any work where a misfire is hard to undo.

## What Gets Blocked

When `/careful` is active, you MUST stop and ask for explicit user confirmation before executing any of the following:

### Database
- `DROP TABLE`, `DROP COLUMN`, `DROP INDEX`, `DROP SCHEMA`
- `TRUNCATE`
- `DELETE FROM` without a `WHERE` clause
- `ALTER TABLE ... DROP ...`
- Any migration that is **not** wrapped in a transaction or uses `IF EXISTS` guards

### Git
- `git push --force` or `git push -f`
- `git reset --hard`
- `git checkout -- .` or `git restore .` (discards uncommitted changes)
- `git branch -D` (force-delete branch)
- `git rebase` on a shared/published branch

### Filesystem
- `rm -rf` on any directory
- Overwriting a file that has not been read in this session

## How to Use

Invoke at the start of any session involving risk:

```
/careful
```

Claude will confirm: **"Safety shield active. I will ask before any destructive operation."**

## How to Disable

`/careful` lasts for the duration of the session only. Start a new session to reset.

If you explicitly want to proceed with a blocked action, say: **"I confirm, proceed"** and Claude will execute it once.

## Pairing Suggestions

- Always invoke `/careful` before running a new Supabase migration against production.
- Pair with `/plan` sessions that include a "Schema/Env Migration" task.
- Use during debugging sessions on live data.
