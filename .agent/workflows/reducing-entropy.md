---
description: Spawn a fresh-eyes adversarial subagent to critique newly written code for bloat, premature abstraction, and DRY violations. Run during /audit on all files modified in the current phase.
---

# /reducing-entropy

Spawns a fresh subagent with no knowledge of why the code was written — only what it is. That subagent acts as a skeptical senior reviewer and critiques until findings degrade to nitpicks.

Use during `/audit` after the test suite passes, before the UX check.

## The Process

### Step 1: Collect Changed Files
Identify all files created or modified in the current phase (use `git diff --name-only HEAD~N` where N is the number of commits in the phase, or check the plan's task list).

### Step 2: Spawn the Adversarial Subagent
Launch a subagent with **only** the changed files as context — no plan doc, no conversation history, no justification. Give it this prompt:

> "Review these files as a senior engineer seeing them for the first time. Identify: (1) code that solves a problem that doesn't exist yet (YAGNI), (2) duplicated logic that should be a shared utility (DRY), (3) abstractions introduced for a single use case (premature abstraction), (4) functions or components doing more than one job (single responsibility), (5) anything that will be confusing to the next person reading this. Be specific — name the file and line. Do not praise what's working."

### Step 3: Triage Findings
Categorize the subagent's output into three buckets:

| Bucket | Definition | Action |
|---|---|---|
| **Blocking** | Violates DRY, YAGNI, or SRP in a way that will cause real future pain | Fix before proceeding |
| **Improvement** | Valid point, non-urgent | Log to `BUGS.md` as Low DEBT |
| **Nitpick** | Style, naming preference, minor | Acknowledge, skip |

### Step 4: Fix Blocking Issues
Apply fixes. Re-run the test suite to confirm nothing broke.

### Step 5: Iterate (if needed)
If blocking issues were found and fixed, spawn the subagent again with the updated files. Repeat until the subagent's highest-severity finding is an "Improvement" or lower.

### Step 6: Report
Return a one-paragraph summary to the main session:
- How many rounds were needed
- What was fixed
- What was logged to BUGS.md
- What was intentionally left as-is and why

## What This Is NOT

- Not a style linter (use ESLint for that)
- Not a security review (use `/audit` P0 checklist for that)
- Not a test coverage check (the test suite handles that)

## Gotcha

The subagent will sometimes critique intentional simplicity as "missing abstraction." Use judgement: **the right amount of complexity is the minimum needed for the current task**. Three similar lines of code is better than a premature helper function.
