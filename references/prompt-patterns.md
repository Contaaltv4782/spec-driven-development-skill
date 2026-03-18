# Prompt Patterns

Effective prompts for each phase of the SDD workflow. The quality of your specs directly
determines the quality of AI output — vague input = vague output.

---

## Phase 1 — Specify Prompts

### Initial Specification Prompt

```
I want to build: [feature description in plain language]

Generate a spec.md for this feature following the SDD workflow. Use the template in
references/artifact-templates.md.

Requirements:
- No implementation details in the spec (no technology names, no function names)
- Each acceptance criterion must be independently testable
- Mark any ambiguous requirements with [NEEDS CLARIFICATION]
- Explicitly list what is OUT OF SCOPE

Target users: [who uses this feature]
Constraints: [any hard limits — performance, security, platform]
```

### Spec Clarification Prompt

```
Review spec.md at specs/[feature]/spec.md and identify:
1. Acceptance criteria that are ambiguous or not independently testable
2. Missing edge cases for AC-[N]
3. Any implicit assumptions that should be made explicit
4. Scope boundaries that need clarification

Do NOT suggest implementation approaches. Focus on requirements only.
```

### Spec Completeness Check Prompt

```
Act as a product manager reviewing specs/[feature]/spec.md.

Check:
- Are all user stories covered by acceptance criteria?
- Are error scenarios and edge cases specified?
- Are performance requirements stated (if applicable)?
- Are security requirements stated (if applicable)?
- Is the "out of scope" section complete?

Return only gaps and missing items, not a full rewrite.
```

---

## Phase 2 — Plan Prompts

### Technical Plan Generation Prompt

```
Read specs/[feature]/spec.md and generate:
1. plan.md — technical architecture and component breakdown
2. data-model.md — entities, fields, relationships, indexes
3. contracts/[name].md — API endpoints (one file per domain)

Constraints:
- Use existing stack: [list your stack, e.g., TypeScript, Hono, Drizzle, PostgreSQL]
- Do not introduce new dependencies unless justified in plan.md
- Use framework features directly — avoid unnecessary wrapper layers
- Every AC in spec.md must map to at least one component in plan.md
- Follow existing patterns in: [reference file or convention]

Generate files sequentially: plan.md first, then data-model.md, then contracts/.
```

### Plan Review Prompt

```
Review the generated plan at specs/[feature]/plan.md against specs/[feature]/spec.md.

Check:
1. AC coverage: does every acceptance criterion have a corresponding component?
2. Completeness: are all data-model.md entities referenced in plan.md?
3. Contract completeness: does every component that exposes an API have a contract?
4. Over-engineering: are there abstractions that could be replaced with direct framework usage?

Return a gap analysis only — do not regenerate the files.
```

---

## Phase 3 — Tasks Prompts

### Task Breakdown Prompt

```
Read specs/[feature]/plan.md and specs/[feature]/contracts/*.md.

Generate tasks.md with:
- Atomic tasks (one task = one PR or commit)
- Test task before each implementation task (test-first)
- [P] marker for tasks that can run in parallel
- S/M/L size estimates
- Explicit dependencies (by task ID)

Rules:
- No task should modify more than 3 files
- Large [L] tasks must be split unless justified
- Test tasks reference specific ACs from spec.md
- Implementation tasks reference specific sections from plan.md and contracts/
```

### Task Validation Prompt

```
Review specs/[feature]/tasks.md and identify:
1. Tasks missing their paired test task
2. Tasks that are too large (would touch > 3 files)
3. Dependency cycles
4. Implementation tasks before their test tasks

Return only issues found, not a full rewrite.
```

---

## Phase 4 — Implementation Prompts

### Single Task Implementation Prompt

```
Implement TASK-[N]: [task title]

Specification:
- Acceptance criteria: [paste relevant ACs from spec.md]
- Technical design: [paste relevant section from plan.md]
- API contract: [paste from contracts/file.md if applicable]
- Data model: [paste relevant entities from data-model.md if applicable]

Constraints:
- Match API signatures exactly as defined in the contract
- Do not add functionality outside the acceptance criteria
- Do not add abstractions not mentioned in plan.md
- Follow project conventions: [reference CLAUDE.md or conventions]

After implementation, verify:
- [ ] All referenced ACs have test coverage
- [ ] API signatures match contracts exactly
- [ ] No new dependencies introduced without explicit justification
```

### Mid-Implementation Correction Prompt (when AI drifts)

```
Stop. The implementation is drifting from the specification.

Spec says: [paste the relevant AC or contract section]
You implemented: [describe what was generated]

Roll back this change and re-implement following the spec exactly.
Do not use your judgment about what "makes sense" — follow the contract.
```

---

## Phase 5 — Validate Prompts

### Drift Detection Prompt

```
Compare the implementation in [file paths] against:
- Acceptance criteria in specs/[feature]/spec.md
- API contracts in specs/[feature]/contracts/
- Data model in specs/[feature]/data-model.md

Report:
1. ACs with no test coverage
2. Implemented API signatures that differ from contracts/
3. Database fields not in data-model.md
4. Functionality in code not covered by spec.md
5. Missing error handling for error codes defined in contracts/

Format as a checklist. Mark each item PASS or FAIL with evidence.
```

### Post-Implementation Validation Prompt

```
Run a full spec compliance check on the [feature] implementation.

Files to check: [list implementation files]
Spec files: specs/[feature]/spec.md, specs/[feature]/plan.md, specs/[feature]/contracts/

For each AC in spec.md, identify:
- Which test covers it
- Which implementation file satisfies it
- Pass/Fail status

Return a traceability matrix.
```

---

## Multi-Agent Review Pattern

When using subagents for critique (recommended for complex specs):

```
# Critic Agent Prompt
Review specs/[feature]/spec.md from the perspective of [role]:
- QA Engineer: find untestable or ambiguous criteria
- Security Engineer: find missing auth, validation, or data exposure risks
- Database Engineer: find schema issues, missing indexes, or N+1 risks
- Frontend Engineer: find missing states, loading states, or error UX

Return only specific, actionable issues. Do not approve or compliment.
```

Use separate critic agents for each role. Aggregate their feedback before proceeding.
