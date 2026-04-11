---
description: Perform a combined technical and UX audit of newly built code.
---

# /audit

Use this workflow to ensure code is production-ready and aesthetically perfect before merging. This replaces separate code review and UX verification steps.

## 1. Technical Audit (The "Engine" Check)
Inspect the implementation for reliability and performance.
*   **P0 Standards**: Consult `.agent/REFERENCE.md` for the Technical Reliability checklist (Timeouts, Idempotency, etc.).
*   **Test Verification**: Run the test suite. If tests are missing or failing, the audit is **FAILED**.
*   **Test Data Parity**: If any core validation rules (e.g., UUID formats, token shapes, or required fields) were changed during development, visually verify that Vitest `.mockResolvedValue` stubs and test objects reflect the new strict shapes BEFORE running `npm test`, preventing cascading false-positive failures.
*   **Schema Integrity**: Run `npm test tests/infrastructure/schema.test.ts`. If columns are missing or data types have drifted without a migration, the audit is **FAILED**.
*   **Clean Code**: Run `/reducing-entropy` on all files modified in this phase. Do not proceed to the UX audit until the adversarial review returns no Blocking findings. Log any Improvement-level findings to `BUGS.md`.
*   **Filter String Wildcards**: If any new code uses Supabase `.or()` with inline filter strings, verify `ilike` patterns use `*` (not `%`). Example: `url.ilike.*linkedin.com*`. The `%` form silently returns no rows when used inside a filter string.
*   **Hooks-in-Lists**: Grep new list-rendering components for hook calls (`use`) inside `.map()` callbacks. Any hook called inside a loop is a Rules of Hooks violation — extract to a sub-component.
*   **Dead Component Check**: For every new component added this phase, verify it is imported and rendered somewhere. Search for its name — if the only match is its own file, it is not wired in. Treat this as a Critical bug.
*   **Path Sanitation**: Verify no URL-encoded directories (e.g., `%5Bid%5D`) exist in the `src/app` tree. These cause silent routing conflicts.

## 2. UX & Aesthetic Audit (The "Chassis" Check)
Verify the interface and user interaction.
*   **Design Standards**: Consult `.agent/REFERENCE.md` for Visual and Interaction standards (Teal anchor, Spring physics).
*   **Manual Verification**: 
    1. Run `npm run dev`.
    2. Open [http://localhost:3000].
    3. Verify responsiveness, Dark Mode parity, and "The Librarian" brand voice.

## 3. Results & Remediation
*   **Small Fixes**: Correct minor typos, spacing, or color variables immediately.
*   **Blocking Issues**: Add any bugs or UX debt to `BUGS.md` with `/log`.
*   **Audit Status**:
    *   ✅ **PASS**: Specs met, P0s satisfied, Aesthetics verified.
    *   ❌ **FAIL**: Known bugs, P0 violations, or UX drift.

**Next Step**: Once audit is passing, run `/teach-me` followed by `/closeout`.
