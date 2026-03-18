# Workflow Phases — Detailed Reference

Step-by-step execution guide for each SDD phase, including decision points and common traps.

---

## Phase 1 — Specify

### Goal
Produce a `spec.md` that a developer (or AI agent) with no prior context can read and
understand exactly what to build — without needing to ask clarifying questions.

### Step-by-Step

**Step 1.1 — Feature intake**
Ask the user (or gather from issue/PRD):
- What problem does this solve?
- Who uses it?
- What does success look like?
- What are hard constraints (performance, security, compatibility)?

**Step 1.2 — Generate spec.md**
Use the prompt in `references/prompt-patterns.md → Phase 1 → Initial Specification Prompt`.
Place output at `specs/[feature-branch-name]/spec.md`.

**Step 1.3 — Resolve open questions**
All items marked `[NEEDS CLARIFICATION]` must be resolved before proceeding.
Do not assume. Do not defer. Get answers.

**Step 1.4 — Human review**
The human must read and approve `spec.md`. This gate is non-negotiable.
Common review feedback:
- "AC is too vague" → rewrite as Given/When/Then
- "This is actually out of scope" → move to out-of-scope section
- "Missing error case" → add AC for the error case

**Step 1.5 — Branch and commit**
```bash
git checkout -b feature/[feature-name]
git add specs/[feature-name]/spec.md
git commit -m "spec: [feature name] requirements"
```

### Time Distribution
Expect to spend 30–60% of total feature time on Phases 1–2. This is correct.
Reduced execution time more than compensates.

---

## Phase 2 — Plan

### Goal
Translate `spec.md` into a concrete technical blueprint that AI can implement without
inventing solutions. Every design decision must be explicit.

### Step-by-Step

**Step 2.1 — Generate plan.md**
Use the Technical Plan Generation Prompt from `references/prompt-patterns.md`.
Include your stack constraints and existing conventions.

**Step 2.2 — Generate data-model.md**
If the feature touches the database:
- List every entity the feature creates, reads, updates, or deletes
- Define all fields with types and constraints
- Define foreign keys and relationships
- Identify required indexes (think about query patterns)

**Step 2.3 — Generate contracts/**
For each API endpoint, event, or public interface:
- One file per domain (e.g., `contracts/user-api.md`, `contracts/events.md`)
- Define exact request/response shapes
- Define all error codes and when they're returned
- Reference which ACs each endpoint satisfies

**Step 2.4 — Trace to spec**
Every AC in `spec.md` must appear in at least one:
- Component in `plan.md`
- Contract in `contracts/`

If an AC has no technical implementation path, the plan is incomplete.

**Step 2.5 — Human review**
The human reviews `plan.md`, `data-model.md`, and `contracts/`. Particular focus:
- Are contracts complete enough to implement without questions?
- Does the data model handle all the spec's data requirements?
- Are there simpler approaches to any components?

**Step 2.6 — Lock contracts**
Once approved, `contracts/` are **frozen** for the duration of Phase 4.
Changing a contract during implementation is spec drift — plan first, then execute.

**Step 2.7 — Commit**
```bash
git add specs/[feature-name]/plan.md specs/[feature-name]/data-model.md specs/[feature-name]/contracts/
git commit -m "plan: [feature name] technical design"
```

---

## Phase 3 — Tasks

### Goal
Break `plan.md` into granular, sequential tasks small enough for a single AI session
(~30–60 minutes of implementation work).

### Step-by-Step

**Step 3.1 — Generate tasks.md**
Use the Task Breakdown Prompt from `references/prompt-patterns.md`.

**Step 3.2 — Apply test-first ordering**
For every implementation task, there must be a test task immediately preceding it:
```
TASK-003: Write tests for UserRepository.create()   ← test first
TASK-004: Implement UserRepository.create()          ← implementation after
```

**Step 3.3 — Size tasks**
- `S` (< 1 hour): single function, small component, migration
- `M` (1–3 hours): a full repository method + tests, an API route + tests
- `L` (3–6 hours): consider splitting into two `M` tasks

**Step 3.4 — Identify parallelism**
Tasks with no shared dependencies can be marked `[P]` and run simultaneously
(in separate AI sessions or by separate developers).

**Step 3.5 — Human review**
Review `tasks.md` for:
- Tasks that seem too large (will need mid-task context reset)
- Test tasks that don't reference specific ACs
- Implementation tasks that reference "see plan.md" vaguely (add specific section)

**Step 3.6 — Commit**
```bash
git add specs/[feature-name]/tasks.md
git commit -m "tasks: [feature name] task breakdown"
```

---

## Phase 4 — Implement

### Goal
Execute tasks from `tasks.md` with AI constrained by spec artifacts, committing after
each task to maintain clean rollback points.

### Step-by-Step

**Step 4.1 — Session setup per task**
Start a fresh context window for each task. Include:
- The task description from `tasks.md`
- Relevant ACs from `spec.md`
- Relevant section from `plan.md`
- Relevant contract from `contracts/`
- Relevant entities from `data-model.md`

Do NOT paste the entire spec — include only what's relevant to the current task.

**Step 4.2 — Use the single-task implementation prompt**
See `references/prompt-patterns.md → Phase 4 → Single Task Implementation Prompt`.

**Step 4.3 — Verify before committing**
After AI generates code, verify:
- Tests pass (run them, don't trust the AI)
- API signatures match contracts exactly
- No new files outside the task scope were created

**Step 4.4 — Commit after each task**
```bash
git add [files]
git commit -m "feat([scope]): [task title]"
```

Do not batch multiple tasks into one commit. Each task = one commit.
This enables precise rollback if a task introduced drift.

**Step 4.5 — Mark task complete in tasks.md**
```markdown
- [x] **TASK-003** [M] Write tests for UserRepository.create()
```

Commit the updated `tasks.md` with the implementation commit.

### Context Reset Between Tasks
Clear the AI context between tasks to prevent:
- Accumulated assumptions from prior sessions
- The AI "remembering" wrong implementations it wrote earlier
- Conflicting information about how the system works

This is especially important when tasks are implemented across multiple days.

---

## Phase 5 — Validate

### Goal
Verify the implementation satisfies every acceptance criterion and no spec drift occurred.

### Step-by-Step

**Step 5.1 — Generate traceability matrix**
Use the Post-Implementation Validation Prompt from `references/prompt-patterns.md`.
Produces: for each AC, which test covers it, which file implements it, pass/fail.

**Step 5.2 — Run drift detection**
Use the Drift Detection Prompt from `references/prompt-patterns.md`.
Checks API signatures, database schema, and behavior against specs.

**Step 5.3 — Fix drift immediately**
If drift is found:
- Do not "adjust the spec to match the code"
- Fix the implementation to match the spec
- If the spec is genuinely wrong, update spec.md, then plan.md, then implementation
  (never skip the update chain)

**Step 5.4 — Acceptance test run**
Manually walk through each user story from `spec.md` in the running application.
Check each AC as a user, not as a developer.

**Step 5.5 — Archive specs**
After validation passes, move specs to long-term storage:
```bash
mkdir -p .specs/[feature-name]
mv specs/[feature-name]/* .specs/[feature-name]/
```
Keep `.specs/` in version control for future reference and regression analysis.
