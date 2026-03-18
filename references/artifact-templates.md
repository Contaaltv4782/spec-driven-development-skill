# Artifact Templates

Copy-paste templates for each SDD artifact. Remove placeholder comments before committing.

---

## spec.md Template

```markdown
# [Feature Name]

## Overview
<!-- 1-2 sentences describing the feature for a non-technical stakeholder -->

## User Stories

### Primary
As a [role], I want [goal] so that [benefit].

### Secondary (optional)
As a [role], I want [goal] so that [benefit].

## Acceptance Criteria

### AC-1: [Short Title]
Given [initial context]
When [action is taken]
Then [expected outcome]

### AC-2: [Short Title]
Given [initial context]
When [action is taken]
Then [expected outcome]

<!-- Add as many ACs as needed. Each must be independently testable. -->

## Out of Scope
<!-- Explicitly list what this feature does NOT include -->
- [Item 1]
- [Item 2]

## Open Questions
<!-- Use [NEEDS CLARIFICATION] for unresolved items. Resolve before Phase 2. -->
- [NEEDS CLARIFICATION] [Question 1]
- [RESOLVED] [Question 2] → Decision: [answer]

## Non-Functional Requirements
<!-- Performance, security, accessibility, internationalization, etc. -->
- Performance: [e.g., "search results in < 200ms at p95"]
- Security: [e.g., "requires authenticated session"]
```

---

## plan.md Template

```markdown
# Technical Plan: [Feature Name]

## Spec Reference
Implements: `specs/[branch]/spec.md`

## Architecture Overview
<!-- High-level description of the approach. 3-5 sentences max. -->

## Component Breakdown

### [Component 1 Name]
- **Responsibility:** [What it does]
- **Location:** `[file path]`
- **Accepts:** [inputs]
- **Returns:** [outputs]
- **AC Coverage:** AC-1, AC-2

### [Component 2 Name]
- **Responsibility:** [What it does]
- **Location:** `[file path]`
- **Accepts:** [inputs]
- **Returns:** [outputs]
- **AC Coverage:** AC-3

## Technology Choices

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [e.g., DB query] | [e.g., Drizzle ORM] | [e.g., already in stack, type-safe] |

## Integration Points
<!-- External services, APIs, or other features this touches -->
- [System]: [How it's used]

## Out of Scope (Technical)
<!-- Technical boundaries that mirror spec.md out-of-scope -->
- No [X]
- No [Y]
```

---

## data-model.md Template

```markdown
# Data Model: [Feature Name]

## Entities

### [EntityName]
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | uuid | PK, NOT NULL | Primary key |
| [field] | [type] | [constraints] | [description] |
| created_at | timestamp | NOT NULL, DEFAULT now() | |
| updated_at | timestamp | NOT NULL | |

### Relationships
- `[EntityA]` has many `[EntityB]` (via `entity_b.entity_a_id`)
- `[EntityA]` belongs to `[EntityC]`

## Indexes
| Table | Columns | Type | Rationale |
|-------|---------|------|-----------|
| [table] | [col1, col2] | btree | [e.g., lookup by user + date] |

## Constraints
- [e.g., `CHECK (status IN ('active', 'inactive', 'pending'))` on `users`]
```

---

## contracts/[endpoint].md Template

```markdown
# API Contract: [Endpoint Name]

## [HTTP METHOD] [/path/:param]

### Description
[One sentence describing what this endpoint does]

### Authentication
[e.g., "Bearer token required" / "Public" / "Admin role required"]

### Request

**Path Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| [param] | string | yes | [description] |

**Query Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| [param] | string | no | [default] | [description] |

**Request Body:**
```json
{
  "field": "type — description",
  "optionalField?": "type — description"
}
```

### Response

**Success (200 OK):**
```json
{
  "id": "uuid",
  "field": "value"
}
```

**Error Codes:**
| Status | Code | When |
|--------|------|------|
| 400 | VALIDATION_ERROR | [condition] |
| 401 | UNAUTHORIZED | [condition] |
| 404 | NOT_FOUND | [condition] |
| 409 | CONFLICT | [condition] |

### AC Coverage
- AC-1: [How this endpoint satisfies it]
```

---

## tasks.md Template

```markdown
# Task List: [Feature Name]

## Plan Reference
Implements: `specs/[branch]/plan.md`

## Tasks

### Setup

- [ ] **TASK-001** [S] Set up [component/module] skeleton
  - Creates: `[file path]`
  - Depends on: none

### [Component Group]

- [ ] **TASK-002** [M] [P] Write tests for [component]
  - Tests: AC-1, AC-2 from `specs/[branch]/spec.md`
  - Depends on: TASK-001

- [ ] **TASK-003** [M] Implement [component]
  - Contract: `specs/[branch]/contracts/[file].md`
  - Depends on: TASK-002

- [ ] **TASK-004** [S] [P] Write tests for [other component]
  - Tests: AC-3 from `specs/[branch]/spec.md`
  - Depends on: TASK-001

- [ ] **TASK-005** [M] Implement [other component]
  - Contract: `specs/[branch]/contracts/[file].md`
  - Depends on: TASK-004

### Integration

- [ ] **TASK-006** [L] Integration test: full [feature] flow
  - Tests: AC-1 through AC-4
  - Depends on: TASK-003, TASK-005

### Cleanup

- [ ] **TASK-007** [S] Update API documentation
  - Depends on: TASK-006

## Legend
- `[S]` Small — under 1 hour
- `[M]` Medium — 1–3 hours
- `[L]` Large — 3–6 hours (consider splitting)
- `[P]` Parallelizable — can run concurrently with other `[P]` tasks at same level
```
