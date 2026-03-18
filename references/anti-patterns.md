# Anti-Patterns

The most common SDD failure modes, their symptoms, and how to fix them.

---

## Anti-Pattern 1: Spec with Implementation Details

**Symptoms:**
- spec.md mentions specific tables, functions, libraries, or frameworks
- Developers disagree about whether to use PostgreSQL or Redis based on spec
- Changing the database engine requires rewriting the spec

**Example (wrong):**
```markdown
## AC-1
The `/api/users` endpoint should query the `users` table using a JOIN
with the `user_profiles` table and return the result as JSON.
```

**Example (correct):**
```markdown
## AC-1
Given a valid session, when the user requests their profile,
then their full profile information is returned within 200ms.
```

**Fix:** Remove all technology references from spec.md. Move them to plan.md.
The spec describes WHAT and WHY. Plan.md describes HOW.

---

## Anti-Pattern 2: Contracts Modified During Implementation

**Symptoms:**
- `contracts/user-api.md` was "improved" after Phase 4 started
- Frontend and backend have different assumptions about the API shape
- Tests written against the old contract, implementation matches the new one

**The trap:** AI suggests "a better" response format while generating code, and you accept it.
Now the contract is stale and other tasks that depend on it are broken.

**Fix:** Lock `contracts/` after Phase 2 approval. If a contract needs to change:
1. Stop Phase 4
2. Update spec.md (if it affects acceptance criteria)
3. Update plan.md
4. Update contracts/
5. Review affected tasks.md entries
6. Resume Phase 4 from the affected task

Never treat contracts as drafts during implementation.

---

## Anti-Pattern 3: One Context for All Tasks

**Symptoms:**
- AI "remembers" it used a different architecture in TASK-002 and applies it to TASK-008
- Accumulated hallucinations from earlier tasks contaminate later ones
- "You already created this function in the previous message" — but it was wrong

**Fix:** Start a fresh AI context for each task. Include only what's relevant:
- The specific task from tasks.md
- The specific ACs it covers from spec.md
- The relevant contract section
- The relevant plan section

More context ≠ better output. Focused context produces better output.

---

## Anti-Pattern 4: Skipping the Human Review Gates

**Symptoms:**
- Spec approved by the AI that generated it
- Plan not reviewed before tasks were created
- Tasks created from an unapproved plan
- "I'll fix it in the code" — drift accumulates

**The trap:** When you're in flow, the gates feel like friction. They're not — they're the
moment where wrong assumptions cost minutes (in specs) vs. hours (in code).

**Fix:** Human approval of each artifact is mandatory. There are three hard gates:
- Gate 1: spec.md approved before plan generation
- Gate 2: plan.md + contracts/ approved before task generation
- Gate 3: tasks.md reviewed before implementation

Unapproved artifacts propagate errors forward into every subsequent phase.

---

## Anti-Pattern 5: Oversized Tasks

**Symptoms:**
- TASK-007 touches 8 files and takes 4 hours
- AI loses context mid-task and asks what the endpoint signature should be
- Multiple unrelated changes in one commit make rollback difficult

**The trap:** "This is all one logical unit" is almost never true — it's usually several
units that happen to be adjacent in the code.

**Fix:** Split any task that:
- Touches more than 3 files
- Has more than one acceptance criterion
- Would produce a commit diff over 200 lines

A task that can be described in one sentence is the right size.

---

## Anti-Pattern 6: Adjusting the Spec to Match the Code

**Symptoms:**
- "The AI implemented it differently, so I updated spec.md to reflect that"
- Spec.md now describes what was built, not what was wanted
- Future features are planned against a spec that documents past drift

**The trap:** It feels like keeping docs in sync. It's actually destroying the spec's value
as a source of truth.

**Fix:** Code must conform to spec. Never the reverse. When drift is found:
1. Fix the implementation to match the spec
2. If the spec is genuinely wrong (new information emerged), update it explicitly:
   - Write a comment in `decision_log.md` explaining why the spec changed
   - Regenerate the affected plan sections
   - Regenerate affected contracts
   - Update affected tasks
   - Re-implement

This feels slow. It's faster than discovering the problem after 6 more features
were built on top of the drift.

---

## Anti-Pattern 7: Vague Acceptance Criteria

**Symptoms:**
- "The feature works correctly" — no test can be written for this
- "The API is fast" — 10ms or 10s?
- "Users can manage their settings" — create, read, update, delete, or just read?

**Fix:** Every AC must pass the testability test:
- Can you write an automated test that returns pass or fail? If no → rewrite.
- Can two developers independently write the same test? If no → rewrite.
- Does it include a measurable threshold for performance/security criteria? If no → add one.

---

## Anti-Pattern 8: Missing Error Cases in Contracts

**Symptoms:**
- Frontend shows a generic 500 error because the contract didn't define 409 CONFLICT
- Auth errors not handled because the contract said "returns 200"
- Duplicate submission creates two records because the idempotency behavior wasn't specified

**Fix:** For every contract, explicitly define:
- All success responses (200, 201, 204)
- All client error responses (400, 401, 403, 404, 409, 422)
- Idempotency behavior (is this endpoint safe to call twice?)
- Rate limiting behavior if applicable

---

## Anti-Pattern 9: SDD for Trivial Changes

**Symptoms:**
- spec.md created for "fix typo in error message"
- Full 5-phase workflow for a 2-line bug fix
- Team resents SDD because overhead exceeds benefit

**Fix:** SDD has a setup cost. Apply it where the payoff exceeds the cost:
- Features that touch ≥ 3 files: use SDD
- Features that touch auth, DB schema, or public API: always use SDD
- Bug fixes under 30 minutes: skip SDD, fix directly
- Refactors with no behavior change: document intent in a commit message, skip SDD
