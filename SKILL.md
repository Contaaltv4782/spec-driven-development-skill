---
name: spec-driven-development
description: >
  Use when starting features, projects, or refactors with AI coding agents and requirements feel
  informal, incomplete, or drift-prone. Triggers: AI generates code that ignores constraints,
  same prompt produces different implementations across sessions, team lacks shared technical
  understanding, complex features need traceable design decisions, or vibe-coding produces
  unreliable output. Keywords: spec-driven, SDD, specification-first, requirements.md, plan.md,
  tasks.md, design doc, PRD, acceptance criteria, drift detection, AI planning, feature spec.
---

# Spec-Driven Development

## Overview

SDD makes **specifications the source of truth** — code serves specs, not the other way around.
Instead of prompting an AI with vague instructions and hoping for the right output, you author
precise specifications first, then let AI generate code constrained by those specs.

Core principle: *"When specifications drive implementation, pivots become systematic
regenerations rather than manual rewrites."*

## When to Use

Symptoms that signal SDD is needed:
- AI ignores constraints or generates code that doesn't match requirements
- Same prompt produces different implementations across sessions
- Requirements are complex with multiple stakeholders or cross-cutting concerns
- Team needs shared technical understanding before writing code
- Previous "just build it" attempts produced wrong architecture
- Feature touches auth, data model, API contracts, or database schema

**Skip SDD for:**
- One-line fixes, typos, trivial bugs
- Disposable prototypes where requirements will immediately change
- Solo exploration spikes lasting less than a day

## Directory Structure

```
specs/
  [feature-branch-name]/
    spec.md              # Requirements — WHAT and WHY
    plan.md              # Implementation strategy — HOW
    data-model.md        # Data entities, relationships, schemas
    contracts/           # API endpoints, events, interface definitions
    tasks.md             # Atomic executable task list
    research.md          # Optional: context and alternatives considered
    decision_log.md      # Optional: rationale for key decisions
```

## 5-Phase Workflow

### Phase 1 — Specify

**Invoke:** `/sdd:specify [feature description]`

Creates `specs/[feature]/spec.md` containing:
- Feature overview (1–2 sentences, non-technical)
- User stories: `As a [role] I want [goal] so that [benefit]`
- Acceptance criteria: testable, unambiguous, implementation-agnostic
- Explicit out-of-scope boundaries
- Open questions marked `[NEEDS CLARIFICATION]`

**Human gate — review before Phase 2:**
- [ ] Every acceptance criterion is independently testable
- [ ] No implementation details inside the spec (no "use PostgreSQL", no function names)
- [ ] All `[NEEDS CLARIFICATION]` items resolved or explicitly deferred
- [ ] Scope boundaries explicitly listed

See `references/artifact-templates.md` for the `spec.md` template.

---

### Phase 2 — Plan

**Invoke:** `/sdd:plan` (reads `spec.md`)

Creates in `specs/[feature]/`:
- `plan.md` — Technical architecture, component breakdown, technology choices, tradeoffs
- `data-model.md` — Entities, fields, relationships, constraints, indexes
- `contracts/` — API endpoint definitions, request/response schemas, error codes, events
- `research.md` — Optional: alternatives considered, rationale for technology choices

**Human gate — review before Phase 3:**
- [ ] Plan is traceable: every acceptance criterion maps to a plan section
- [ ] Data model covers all entities mentioned in spec
- [ ] API contracts are complete (inputs, outputs, error cases)
- [ ] No unnecessary abstractions (use framework features directly)
- [ ] Technology choices are justified

See `references/artifact-templates.md` for templates.

---

### Phase 3 — Tasks

**Invoke:** `/sdd:tasks` (reads `plan.md` + `contracts/`)

Creates `specs/[feature]/tasks.md`:
- Atomic tasks — each corresponds to a single PR or commit
- Explicit dependencies between tasks
- Parallelizable tasks marked `[P]`
- Each implementation task paired with a test task (test-first)
- Complexity estimate per task: `S` / `M` / `L`

**Human gate — review before Phase 4:**
- [ ] Tasks are small enough for a single AI session (~30–60 min of work)
- [ ] Test tasks appear before their implementation counterparts
- [ ] No task modifies more than 3 files (split if so)
- [ ] Dependencies form a valid DAG (no cycles)

See `references/artifact-templates.md` for the `tasks.md` template.

---

### Phase 4 — Implement

Execute tasks from `tasks.md` sequentially (or in parallel where marked `[P]`).

**AI prompt pattern for each task:**

```
Implement: [task title from tasks.md]

Constrained by:
- Acceptance criteria: specs/[feature]/spec.md → [section]
- Technical design: specs/[feature]/plan.md → [section]
- API contract: specs/[feature]/contracts/[file].md
- Data model: specs/[feature]/data-model.md

Do NOT:
- Add functionality outside the scope of spec.md
- Deviate from the API signatures in contracts/
- Introduce abstractions not in plan.md
```

**Session hygiene:**
- Clear context between unrelated tasks to prevent conflicting information
- Commit after completing each task before starting the next
- If the AI deviates from the contract, correct immediately — do not accumulate drift

See `references/prompt-patterns.md` for full prompt examples.

---

### Phase 5 — Validate

**Invoke:** `/sdd:validate` (compares implementation to `spec.md` + `contracts/`)

Checks for spec drift:
- Do all acceptance criteria have test coverage?
- Do implemented API signatures match `contracts/`?
- Does the database schema match `data-model.md`?
- Are there any fields, endpoints, or behaviors outside the defined scope?

**Drift indicators (immediate action required):**
- Function signatures that differ from `contracts/`
- Database columns not in `data-model.md`
- Acceptance criteria with no corresponding test
- Functionality that appears in code but not in `spec.md`

See `references/quality-gates.md` for CI/CD integration patterns.

---

## Spec Drift: Why It Happens and How to Prevent It

Drift occurs when AI makes "reasonable assumptions" that weren't specified. Compounding drift
across multiple implementation sessions is the #1 failure mode of AI-assisted development.

Prevention:
1. **Specificity beats verbosity** — "returns 404 when resource not found" beats 3 paragraphs
2. **Lock contracts before implementation** — never modify `contracts/` during Phase 4
3. **One context per task** — a fresh AI session per task eliminates accumulated assumptions
4. **Commit gates** — only proceed to next task after current task passes tests

See `references/anti-patterns.md` for detailed failure modes.

---

## Reference Index

| Need | File |
|------|------|
| Templates for spec.md, plan.md, tasks.md | `references/artifact-templates.md` |
| Prompts for each phase interaction | `references/prompt-patterns.md` |
| Detailed phase-by-phase instructions | `references/workflow-phases.md` |
| Review checklists and CI/CD integration | `references/quality-gates.md` |
| Multi-agent orchestration patterns | `references/ai-agent-patterns.md` |
| Common failure modes and fixes | `references/anti-patterns.md` |
| One-page cheat sheet | `references/quick-reference.md` |
| Topic navigation | `references/INDEX.md` |
