---
description: Create a detailed, TDD-first implementation plan from an approved design.
---

# /plan

Use this workflow once a design has been approved to create a bite-sized, executable path forward.

## Core Principles
- **TDD-First**: Every task must start with a failing test.
- **Architectural Rigor**: Apply Clean Architecture, SOLID, and Domain-Driven Design (DDD) to ensure maintainability and high quality.
- **Bite-sized Granularity**: Each task should take 2-5 minutes to implement.
- **Zero Ambiguity**: Use exact file paths and specify line ranges when modifying.
- **Goals & Success Criteria**: Every phase must have measurable outcomes.
- **Deep Planning**: For high-stakes or complex features, use the `gepetto` skill to perform exhaustive research and specs synthesis before writing the plan.
- **Visual Clarity**: Use the `c4-architecture` skill to define system boundaries and container relationships for architectural changes.

## Process (Before Generating Plan)

**IF FRONTEND CHANGES ARE REQUIRED:**
1. **Mockup First**: You MUST create a single-file HTML prototype (e.g., `public/prototype_<feature_name>.html`) to mimic functionality and aesthetics.
   - Use standard HTML/CSS/JS (no build step required).
   - Ensure it matches `BRAND_GUIDELINES.md`.
   - Mimic interactions (modals, transitions, states) using vanilla JS.
2. **User Validation**: Present this mockup to the user for "look and feel" approval.
3. **Iterate**: Refine the mockup until the user says "Yes, build this."
4. **Data First Check**: Before planning UI, define the data source.
   - WHERE does the data come from? (API? Local Storage? File?)
   - IF the source doesn't exist, create a task to build it.
   - *Tip*: "Mocking" in the UI component is technical debt. Mock at the source/API level instead.
5. **Infrastructure Safety Checks**:
    - **Remote Schema Verification**: Verify that every database column referenced in new code (e.g., `api_token`) has actually been applied to the live Supabase project. Run `npm test tests/infrastructure/schema.test.ts` and `node scripts/status-quovadis.js` to ensure the live DB matches expectations.
    - **Environment Parity**: Ensure your local `.env.local` has the exact same secrets (Client IDs, Secrets) as Vercel before testing begins.
    - IF columns are missing, keys are missing, or data types are wrong, create a **Safety/Migration** task as the first bite-sized task.
6. **Reliability Pre-flight**: Before finalizing the plan, map all external interactions.
    - [ ] **Timeout Mapping**: Identify every `fetch` or `supabase` call and assign a timeout (10s for standard UI actions, 30s for heavy AI/TTS operations).
    - [ ] **Error Toasts**: Ensure every `catch` block includes both a `toast.error` for the user and a `console.error` for technical debugging.
    - [ ] **Local Service URL**: Never use `localhost` for local service connections (Ollama, Redis, Postgres direct). Always use `127.0.0.1` (IPv4 explicit). On macOS 13+, `localhost` may resolve to `::1` (IPv6) while the service binds only to `127.0.0.1`, causing silent connection failures.
    - [ ] **Live-Service Test Gate**: Any test that calls a real external service (local or remote) must be gated behind `describe.skipIf(!process.env.MY_TEST_FLAG)` so CI passes without that service running.
7. **Component Selection Check**: Before choosing a UI component, verify default behaviour that may need overriding.
    - [ ] **shadcn Default Override**: If using `<DialogContent>`, `<SheetContent>`, or any shadcn wrapper that injects responsive classes (e.g., `sm:max-w-lg`), any custom size class must use a matching breakpoint prefix (`sm:max-w-4xl`, not just `max-w-4xl`). Plain utility classes lose to responsive defaults in Tailwind's cascade.
    - [ ] **Radix Select Positioning**: `<Select.Content>` without `position="popper"` ignores trigger width. Always add `position="popper" sideOffset={4}` and `className="w-[var(--radix-select-trigger-width)]"` for dropdowns that should align to their trigger.
8. **Conventions Check**: Before writing the plan, read 1-2 existing files in each category you'll be creating (test, component, API route) to learn how the codebase already does things. Document key conventions as constraints in the plan:
     - [ ] **Auth Architecture Guard**: A mandatory check to ensure the **Sign In** page and the **API routes** are using the exact same provider (no splitting between NextAuth and Supabase). Review Phase History and existing wrappers.
     - [ ] **Test Style**: What assertion library? (e.g., `.toBeDefined()` vs `.toBeInTheDocument()`)? What mock patterns?
     - [ ] **File Patterns**: How are imports structured? Named exports or defaults? Where do types live? (Check: Is it `proxy.ts` or `middleware.ts` for the current Next.js version?)
     - [ ] **Error Handling**: What's the existing `catch` pattern? Toast + console, or something else?
     - [ ] **Reuse Audit**: For every new function the plan calls for, search the codebase first. If an existing function covers ≥80% of the need, the plan task must say "call `X`, don't re-implement." State "No existing equivalent found" explicitly if the search came up empty. Agents will not look for helpers unless told to.
9. **Plan**: ONLY then, proceed to write the Implementation Plan below, referencing the approved mockup as the "Spec".

## Plan Structure (MUST include these sections)

1. **Header**
   - **Goal**: One sentence describing what this builds.
   - **Architecture**: 2-3 sentences about the approach (e.g., "Dependency Inversion for testing," "Domain Layer isolation").
   - **Design Patterns**: Specify if using Factory, Observer, Repository, etc.
   - **Tech Stack**: Libraries or frameworks involved.

2. **Implementation Phases**
   - Break work down into logic phases (e.g., Phase 1: Data Schema, Phase 2: Core Logic).
   - For each phase, list **Success Criteria** using checkboxes.

3. **Bite-Sized Tasks**
   For each task, provide:
   - **Files**: Create: `path/to/new.ts`, Modify: `path/to/old.ts:L10-20`.
   - **Step 1: Write failing test**: Provide the minimal test code.
   - **Step 2: Verify failure**: Specify the command and expected error.
   - **Step 3: Implement minimal code**: Provide the smallest implementation.
   - **Step 4: Verify pass**: Specify command and expected output.
   - **Step 5: Commit**: Provide the bash command and atomic message.

4. **Technical Debt Strategy**
   - Identify any known shortcuts or "hacks" being introduced to meet the goal.
   - List them explicitly so they can be added to `BUGS.md` or a technical debt tracker if the plan is approved without addressing them immediately.

5. **Production & Design Standards (P0)**
   - Before finalizing the plan, ensure the design satisfies the standards in **.agent/REFERENCE.md**.
   - **Timeout Mapping**: Identify every `fetch` or `supabase` call and assign a timeout (10s for standard UI, 30s for AI) as per REFERENCE.md.
   - **Error Handling**: Plan for `toast.error` + `console.error` on every async path.
   - **Loading States**: Every new route MUST include a `loading.tsx` with skeleton loaders.

6. **For UI Features**
   - Check for `.interface-design/system.md`
   - If exists: Load and apply established patterns in implementation plan
   - If not: Include design system creation as a task in the plan

## Persistence
- Save the plan directly to the project repository path `docs/plans/YYYY-MM-DD-<feature-name>.md`. Do NOT use internal AI artifact systems for this document.
- Ask the user: "Ready to start building? Use `/build`."

## Phase Completion Requirements

A phase is NOT complete until the following workflow sequence is executed:

1. **`/build`** - Execute the implementation plan
2. **`/audit`** - Perform combined technical and UX verification
3. **`/closeout`** - Document and commit

Skipping `/audit` is not permitted. The phase remains open until BUGS.md shows zero active items for that phase.

**Internal Note**: Use the `test-driven-development` skill during implementation. For complex architecture, use `c4-architecture`. For deep research, use `gepetto`.
