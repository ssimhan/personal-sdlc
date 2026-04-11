---
description: Resolve ALL bugs and technical debt before phase completion. No exceptions.
---

# /fix

Use this workflow to eliminate all bugs and debt identified during **/audit**. Nothing carries forward to the next phase.

## Core Principles
- **Zero Carryover**: Every bug and debt item is resolved before closeout. No deferrals.
- **Decompose to Conquer**: Large fixes get broken into atomic sub-tasks.
- **Regression Prevention**: Every fix includes a test proving the bug is dead.
- **Scope Discipline**: Fix what's documented. New discoveries go to BUGS.md, then get fixed.

## The Process

### 0. Pre-Flight Lint Audit
Before touching any file, capture the full error scope in one pass:
```bash
npm run lint 2>&1 | grep "error"
```
- **List all errors by file.** Fix ALL errors in a single file before moving to the next.
- Do NOT fix one error, re-lint, discover a new one, repeat. **Batch your changes.**
- After editing any file, grep for duplicate imports: `grep -n "^import" <file> | sort | uniq -d`
- This prevents the "whack-a-mole" lint loop where each fix introduces a new error.

### 1. Inventory
- Read `BUGS.md` to inventory ALL active bugs and debt.
- Count total items. This is your target: reduce to zero.
- Create TodoWrite list with all items.

### 2. Decomposition
For each item, assess complexity:
- **Simple** (< 30 min): Fix directly.
- **Medium** (30 min - 2 hrs): Break into 2-3 sub-tasks.
- **Complex** (> 2 hrs): Break into atomic units, each independently testable and commitable.

**Dark Mode / Token Audit** is a batch refactor, not a feature bug. When you see a `BUG-*-UI-*` item about hardcoded colors:
1. Read `src/app/globals.css` and build an explicit replacement map (hardcoded hex → semantic CSS token).
2. Run `grep -rn "bg-\[#\|text-\[#\|border-\[#" src/` to find all instances.
3. Apply all replacements in a single commit per file group — do not mix color replacements with feature changes.

Document the breakdown in TodoWrite before starting fixes.

### 3. Fix Loop (for each item/sub-task)
- **Reproduce**: Confirm the bug exists (write a failing test).
- **Isolate**: Identify root cause, not just symptoms.
- **Fix**: Apply minimal, targeted change.
  - *Tip*: When ensuring functionality on legacy CommonJS (`require`) files, prefer `vi.spyOn` over `vi.mock` to avoid module interop issues.
- **Verify**:
  - Failing test now passes
  - Full test suite passes (no regressions)
- **Update BUGS.md**: Mark as Fixed with brief note.
- **Commit**: Atomic commit per fix (e.g., `fix(area): resolve BUG-XXX - description`).

### 4. Debt Resolution
Same loop as bugs, but with refactoring mindset:
- Ensure tests cover refactored code paths.
- Break large refactors into sequential, stable states.
- Each commit leaves codebase in working state.

### 5. Discovery During Fixes
When fixing reveals NEW issues:
- Add to BUGS.md immediately.
- Add to TodoWrite list.
- Fix before proceeding to closeout.

### 6. Completion Gate
Before proceeding to `/closeout`, ALL must be true:
- [ ] Active Bugs = 0
- [ ] Active Debt/Features = 0
- [ ] All tests passing
- [ ] No new TODOs/FIXMEs introduced
- [ ] BUGS.md shows all items resolved

## Handling Large Fix Sessions
If total fix work exceeds 4 hours:
- Group related fixes into logical batches.
- Complete and commit each batch.
- Take breaks between batches to maintain quality.
- Never rush—rushed fixes create new bugs.

## Anti-Patterns
- Deferring items to "next phase"
- Combining multiple fixes in one commit
- Skipping tests because "it's a simple fix"
- Expanding scope during fix ("while I'm here..." -> add to BUGS.md instead)
- Marking items as "won't fix" without user approval

**Next Step**: Once all items are resolved, use `/closeout` to document and commit.
