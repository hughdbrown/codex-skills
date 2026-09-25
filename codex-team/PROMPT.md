Use the text below as the basis for a SKILL.md (or skill) for dividing the work between Sol and Luna agents:
```
# Sol + Luna Agent Workflow

**Use Sol as the technical lead, planner, integrator, and reviewer; use Luna as the high-volume implementation and verification worker.**

This can yield materially more completed web-development work than assigning everything to Sol. It does **not** achieve a literal 20× gain, because delegation adds planning, shared-context, review, integration, and rework overhead. For properly decomposed tasks, however, it can realistically produce roughly **2–5× more useful completed work per Sol-equivalent budget** than all-Sol execution.

> The key idea: delegate *bounded, verifiable deliverables* to Luna—not vague, end-to-end features.

---

## Core Cost Model

Luna is approximately 20× cheaper than Sol per equivalent token category, based on public API-equivalent pricing.

| Model | Input per 1M tokens | Output per 1M tokens | Relative cost |
|---|---:|---:|---:|
| Luna | $0.10 | $0.50 | 1× |
| Sol | $2.00 | $10.00 | 20× |

The actual multi-agent cost is:

\[
\text{Total cost} =
\text{Sol planning} +
\sum \text{Luna worker runs} +
\text{Sol review/integration} +
\text{rework and correction loops}
\]

### Expected Cost Outcomes

| Workflow | Approximate cost | Expected result |
|---|---:|---|
| Sol performs an entire medium feature end to end | 1.0× Sol baseline | Highest coherence, simplest operationally |
| Sol plans; Luna implements isolated sub-tasks; Sol integrates | 0.2–0.5× Sol baseline | Usually the best throughput/quality tradeoff |
| Sol delegates poorly scoped, overlapping work to multiple Lunas | 0.7–1.2× Sol baseline | Merge conflicts, rework, unclear ownership |
| Luna works without Sol review or deterministic tests | Low initial cost, high downstream risk | Fast output, unreliable correctness |

> Do not assume “Luna is 20× cheaper” means an entire feature becomes 20× cheaper. Every agent needs context, tool usage, test runs, and a handoff. The realistic gain comes from assigning Luna work that Sol would otherwise spend expensive tokens performing mechanically.

---

## Recommended Division of Labor

| Stage | Primary model | Responsibility | Why |
|---|---|---|---|
| Repository reconnaissance | Luna, with Sol escalation | Inspect modules, locate tests, trace data flow, identify conventions | Cheap parallel evidence gathering |
| Requirements interpretation | Sol | Resolve ambiguity, establish scope and non-goals | Requires judgment about intent |
| Architecture and design | Sol | Choose patterns, boundaries, data flow, authorization approach, migration strategy | Bad decisions here are costly to unwind |
| Task decomposition | Sol | Split work into isolated task cards with contracts and testable acceptance criteria | Determines whether delegation succeeds |
| Interface contracts | Sol | Define API schemas, domain types, shared interfaces, invariants, migration sequencing | Prevents workers from making incompatible assumptions |
| Bounded implementation | Luna | Implement one isolated unit of work in a worktree | High-throughput execution |
| Focused local testing | Luna | Run typechecks, linting, unit tests, and task-specific integration tests | Worker has immediate knowledge of its diff |
| Diff review | Sol | Check correctness, consistency, scope, architecture, and security implications | Preserves system coherence |
| Localized repair | Luna | Address explicit review findings or failing tests | Efficient when the defect is concrete |
| Cross-cutting repair | Sol | Resolve design errors, interface mismatches, or multi-module failures | Requires broader context and judgment |
| Integration and merge | Sol | Resolve conflicts, validate contracts, merge into integration branch | Prevents divergent worker assumptions |
| Final acceptance | Sol plus CI | Verify acceptance criteria, regression tests, security constraints, and end-to-end behavior | “Looks right” is not sufficient |

---

## Sol’s Responsibilities

Sol should act as a **technical lead**, not merely a dispatcher.

Sol owns work that requires broad context, difficult judgment, system-level reasoning, or cross-task coordination.

### Sol should own

- Interpreting the requested feature or ticket
- Identifying ambiguity and asking for clarification when necessary
- Defining scope, non-goals, and expected user-visible behavior
- Choosing frontend, backend, database, and service boundaries
- Designing domain models and data lifecycle rules
- Designing authorization and security-sensitive behavior
- Defining API contracts and shared TypeScript, schema, or OpenAPI types
- Selecting migration strategies and backward-compatibility plans
- Identifying shared files that must not be modified concurrently
- Breaking a feature into small tasks with clean ownership
- Reviewing Luna diffs for correctness, scope, and maintainability
- Integrating independently completed changes
- Resolving merge conflicts and cross-task contract mismatches
- Running final end-to-end verification
- Approving or rejecting completed work

### Sol should avoid spending time on

- Boilerplate CRUD code with an explicit local pattern
- Straightforward UI components with fixed props and behavior
- Mechanical type/schema propagation
- Repetitive test case expansion
- Narrow lint or type-error cleanup
- Predictable file-local refactors
- Documentation generated from an existing contract
- Repository reconnaissance that can be partitioned into clear search tasks

> Sol’s scarce capability should be used on **judgment, decomposition, integration, and difficult debugging**. Do not spend it on work that a well-scoped Luna task can complete with tests.

---

## Luna’s Responsibilities

Luna should operate as a **bounded implementation worker**.

Luna performs best when the task has a narrow scope, clear ownership, explicit inputs and outputs, stable constraints, and deterministic verification.

### Luna should own

- Implementing a component with a defined interface
- Adding a specific endpoint using an existing project pattern
- Updating a schema after a contract is already decided
- Writing tests for predefined acceptance cases
- Adding fixtures, mocks, factories, or test data
- Fixing localized test failures with clear failure output
- Updating generated types or API clients
- Performing targeted codebase reconnaissance
- Reviewing one narrow dimension of a change
- Making mechanical refactors with explicit file boundaries
- Resolving lint and type errors
- Adding validation rules already specified by Sol
- Implementing a database migration whose intended data model is already decided
- Building a page or form against an established API contract

### Luna should not own by default

- Overall application architecture
- Security model decisions
- Authorization policy design
- Ambiguous product requirements
- Domain-model redesign
- Large cross-cutting refactors
- Multiple overlapping changes in shared files
- Production incident diagnosis without a specific hypothesis
- Broad visual-design decisions without a precise design system
- Final merge approval
- Release approval

> Luna should be given enough context to implement its assigned contract—not enough authority to reinterpret the product or redesign the system.

---

## Good Task Boundaries

A Luna task should be understandable as a small engineering unit with an objectively checkable result.

### Good Luna task

> Add the `expires_at` field to the organization invitation persistence model and create the corresponding migration.
>
> **Allowed files:** `db/migrations/*`, `src/db/schema.ts`, `src/models/invitation.ts`
>
> **Do not modify:** API routes, authorization logic, frontend files, email-delivery code
>
> **Requirements:**
>
> - Add nullable `expires_at` with the established timestamp convention
> - Preserve compatibility with existing records
> - Update model serialization if required by existing conventions
> - Add or update migration tests if this repository contains migration tests
>
> **Verification:**
>
> ```bash
> pnpm test db
> pnpm typecheck
> ```
>
> **Done when:**
>
> - The migration applies cleanly to a fresh database
> - Existing invitation records remain readable
> - Required checks pass
> - A commit is created with a concise summary

This task is appropriate for Luna because the intended data-model decision is already made, file ownership is clear, and verification is objective.

### Bad Luna task

> Implement organization invitations across the app.

This is not a good worker task because it combines:

- Database schema design
- API behavior
- Authorization
- Email delivery
- Frontend state and error handling
- Routing
- UX decisions
- Tests
- Migration behavior
- Integration concerns

Sol should convert this feature into multiple bounded tasks before delegating.

---

## Example Feature Breakdown

Consider a feature request:

> Allow organization administrators to invite users by email, assign a role, show pending invitations, and revoke invitations.

Sol should create a feature specification and then partition the work.

### Sol-owned feature plan

```text
Feature: Organization invitations

Goals:
- Administrators can invite a user by email
- Invitations have an assigned role and expiration
- Pending invitations are visible in organization settings
- Administrators can revoke pending invitations

Non-goals:
- No bulk invitation import
- No invitation resend UI in this iteration
- No cross-organization transfer flow

Security invariants:
- Only organization administrators may create or revoke invitations
- Invite tokens must not be exposed in API list responses
- Revoked or expired invitations cannot be accepted
```

### Luna task decomposition

| Task | Owner | Boundary |
|---|---|---|
| Inspect current organization, membership, and email patterns | Luna | Reconnaissance only; no production edits |
| Add invitation schema and migration | Luna | Database files only |
| Add invitation repository methods | Luna | Repository/data-access layer only |
| Implement create-invitation endpoint | Luna | Route/controller/service files only; use Sol-defined contract |
| Implement list-pending-invitations endpoint | Luna | Route/controller/service files only |
| Implement revoke-invitation endpoint | Luna | Route/controller/service files only |
| Add frontend invitation list component | Luna | Frontend component files only |
| Add invitation form component | Luna | Frontend component files only |
| Add API client methods and typed response models | Luna | Client/types files only |
| Add endpoint tests | Luna | Test files only |
| Add end-to-end browser test | Luna | E2E files only |
| Integrate, review, resolve conflicts, validate security model | Sol | All affected layers |

### Task dependencies

```text
Reconnaissance
      |
      v
Sol contract and schema decision
      |
      +--------------------+
      |                    |
      v                    v
Database migration     API client type contract
      |                    |
      v                    |
Repository methods     |
      |                    |
      +---------+----------+
                |
                v
      API endpoints and tests
                |
                v
      Frontend form and list UI
                |
                v
        Browser end-to-end test
                |
                v
     Sol integration and final review
```

> Parallelize only tasks with stable interfaces and no overlapping mutable files.

---

## Task Card Template

Store each delegated task as a file or structured record, such as `tasks/TASK-042.md`.

````md
# TASK-042: Add pending invitation list endpoint

## Goal

Implement an authenticated endpoint that returns pending invitations for one organization.

## Scope

Allowed files:

- `src/api/organizations/invitations.ts`
- `src/services/invitations.ts`
- `src/api/organizations/invitations.test.ts`

Do not modify:

- Database migrations
- Frontend files
- Authentication middleware
- Shared authorization policy
- Email delivery code

## Contract

Route:

```http
GET /api/organizations/:organizationId/invitations
```

Response:

```ts
type PendingInvitation = {
  id: string;
  email: string;
  role: "member" | "admin";
  createdAt: string;
  expiresAt: string | null;
};

type ListPendingInvitationsResponse = {
  invitations: PendingInvitation[];
};
```

## Requirements

- Require authenticated user
- Require organization administrator role
- Return only pending, non-revoked invitations
- Do not expose invitation tokens
- Follow existing error-response conventions
- Use the existing repository abstraction

## Acceptance Criteria

- Non-admin callers receive the established forbidden response
- Invitations from other organizations are not returned
- Expired and revoked invitations are excluded
- Response matches the defined type contract
- Existing endpoint tests remain green

## Verification

Run:

```bash
pnpm test src/api/organizations/invitations.test.ts
pnpm typecheck
pnpm lint
```

## Handoff Format

Return:

- Summary of implementation
- Files changed
- Commands run and results
- Tests added or changed
- Known limitations or questions
- Commit SHA
````

---

## Required Luna Handoff

Every Luna task should produce a concise, structured handoff.

````md
# TASK-042 Handoff

## Status

Completed

## Summary

Implemented the pending invitation listing endpoint for organization administrators.

## Files Changed

- `src/api/organizations/invitations.ts`
- `src/services/invitations.ts`
- `src/api/organizations/invitations.test.ts`

## Verification

```text
pnpm test src/api/organizations/invitations.test.ts
PASS

pnpm typecheck
PASS

pnpm lint
PASS
```

## Tests Added

- Returns only pending invitations
- Excludes revoked invitations
- Excludes expired invitations
- Rejects non-admin organization members
- Prevents cross-organization access

## Known Limitations

- No pagination was added because the endpoint contract did not specify it.

## Commit

```text
abc1234 Add pending organization invitation endpoint
```
````

> Do not accept “implemented successfully” as evidence. Require a diff, test output, commit SHA, and explicit uncertainty reporting.

---

## Recommended Repository Artifacts

Use files as persistent, inspectable state rather than relying on chat history.

| Artifact | Purpose | Owned by |
|---|---|---|
| `AGENTS.md` | Repository commands, coding rules, architecture, conventions, constraints | Human + Sol |
| `SPEC.md` | Feature-level requirements, non-goals, acceptance criteria | Sol |
| `PLAN.md` | Dependency graph, task sequence, shared-file ownership | Sol |
| `contracts/` | Typed API contracts, schemas, interface definitions | Sol |
| `tasks/TASK-###.md` | Bounded worker assignments | Sol |
| `PROGRESS.md` or `tasks.json` | Task status, owners, commits, test evidence | Sol and orchestration layer |
| `docs/decisions/` | Architecture decision records for significant choices | Sol |
| CI configuration | Deterministic verification gates | Human + Sol |

### Suggested repository layout

```text
.
├── AGENTS.md
├── SPEC.md
├── PLAN.md
├── PROGRESS.md
├── contracts/
│   ├── invitations.ts
│   └── organization.ts
├── tasks/
│   ├── TASK-001-recon.md
│   ├── TASK-002-schema.md
│   ├── TASK-003-create-endpoint.md
│   ├── TASK-004-list-endpoint.md
│   └── TASK-005-ui.md
├── docs/
│   └── decisions/
└── src/
```

---

## Agent Execution Loop

Use a deterministic workflow rather than an open-ended conversation.

```text
1. Sol inspects the feature request and repository context.
2. Sol writes or updates SPEC.md.
3. Sol defines contracts and architectural boundaries.
4. Sol creates small, testable TASK files.
5. Luna workers execute isolated tasks in separate worktrees.
6. Each Luna worker runs scoped verification and creates a commit.
7. Sol reviews each diff against the task card and contracts.
8. Sol sends localized defects back to Luna as correction tasks.
9. Sol merges accepted worker commits into an integration branch.
10. Sol runs integration checks, full tests, and browser tests as needed.
11. Sol resolves cross-task issues or escalates them to the human.
12. Sol marks the feature accepted only when acceptance criteria pass.
```

### Correction loop

```text
Luna implementation
      |
      v
Scoped tests pass?
      |
  no  +--------------------------> Luna fixes explicit failures
      |
 yes  v
Sol reviews diff and handoff
      |
  no  +--------------------------> Luna receives focused correction task
      |
 yes  v
Merge to integration branch
      |
      v
Integration and E2E checks pass?
      |
  no  +--------------------------> Sol diagnoses:
                                  - Local repair: Luna
                                  - Design/cross-cutting repair: Sol
      |
 yes  v
Feature accepted
```

---

## Parallelism Rules

Parallelize **investigation** more aggressively than code modification.

### Safe to parallelize

- Codebase reconnaissance in distinct packages
- Test discovery and coverage-gap analysis
- Independent frontend components with fixed interfaces
- Separate API endpoints with no shared-file overlap
- Isolated database migrations after Sol defines the schema
- Documentation updates
- Independent review passes
- Test writing for separate modules
- Lint/type cleanup in non-overlapping file groups
- Fixture and mock generation

### Parallelize cautiously

- Frontend and backend implementation that depend on a fixed API contract
- Multiple endpoints sharing a service or repository layer
- Database schema and persistence code
- UI components sharing state-management files
- Cross-package refactors
- Performance work

### Do not parallelize by default

- Architecture design
- Domain-model redesign
- Authorization model changes
- Shared configuration files
- Shared routing tables
- Shared API schemas while the contract is still evolving
- Large cross-cutting refactors
- Production incident mitigation
- Release branch merge and deployment preparation

> If two workers may edit the same file, either serialize the work, assign one worker sole ownership, or have Sol own the shared file.

---

## Worktree Strategy

Use a separate Git worktree or isolated branch for each Luna worker that writes code.

```bash
git worktree add ../project-task-042 -b agent/task-042
git worktree add ../project-task-043 -b agent/task-043
git worktree add ../project-task-044 -b agent/task-044
```

Each worker should:

1. Work only in its assigned worktree.
2. Modify only the files authorized by its task card.
3. Run the specified verification commands.
4. Commit its completed work.
5. Return the commit SHA and handoff report.
6. Never merge directly into the integration or main branch.

Sol should:

1. Inspect the worker’s diff.
2. Review test evidence.
3. Cherry-pick or merge the accepted commit into an integration branch.
4. Resolve conflicts centrally.
5. Run integration checks after each logical merge group.

This prevents one worker’s unfinished changes from contaminating another worker’s environment and makes rollback straightforward.

---

## Review Policy

### Sol review checklist

Before accepting a Luna implementation, Sol should check:

- Does the diff satisfy the task card exactly?
- Did the worker stay within its allowed file boundaries?
- Does the implementation honor the declared contract?
- Are validation and error behavior consistent with repository conventions?
- Are authorization and tenancy boundaries preserved?
- Are database changes safe and reversible where required?
- Are tests meaningful, rather than merely increasing coverage?
- Does the worker introduce an unnecessary abstraction?
- Does the diff create future integration problems?
- Are errors, loading states, retries, and edge cases handled appropriately?
- Does the change create security, privacy, or data-loss risk?
- Did all required checks actually pass?

### Automatic merge policy

Initially, require Sol review for every Luna-authored production change.

Later, consider automatic merge only for highly constrained task classes:

- Generated client or type updates
- Formatting-only changes
- Fully mechanical codemods
- Documentation generated from committed contracts
- Isolated test-fixture additions
- Dependency lockfile updates accompanied by passing CI
- Simple lint/type-error repairs with no behavior change

> CI is necessary but not sufficient. A passing test suite proves only what the suite checks.

---

## When to Escalate to Sol

Stop retrying with Luna and return the task to Sol if any of the following occurs:

- The worker fails the same task twice.
- The task requires modifying files outside its declared ownership boundary.
- The worker reports an ambiguous contract or missing requirement.
- Tests reveal a flaw in the underlying architecture.
- Two worker results conflict on an interface or behavior.
- A shared file becomes the source of repeated merge conflicts.
- The work involves authorization, billing, encryption, security, data deletion, or privacy-sensitive behavior.
- The implementation requires a broad refactor to complete safely.
- The worker cannot make progress without redesigning a domain model.
- The task’s acceptance criteria cannot be expressed as deterministic tests or observable behavior.

Sol should decide whether to:

- Clarify the specification.
- Redesign the task boundary.
- Make the architectural decision itself.
- Serialize previously parallel work.
- Implement the difficult portion directly.
- Escalate the product or engineering decision to the human.

---

## Prompting Rules

### Sol coordinator prompt rules

Sol should be instructed to:

- Prefer small, independently testable task cards.
- Define file ownership and forbidden files.
- Specify contracts before assigning implementation.
- Identify dependencies and shared-file hazards.
- Avoid delegating ambiguous product decisions.
- Use Luna for research, bounded code changes, tests, and localized repairs.
- Review actual diffs and test evidence.
- Keep plans concise and operational.
- Preserve architectural consistency.
- Stop delegation when a task becomes cross-cutting or unclear.

### Luna worker prompt rules

Luna should be instructed to:

- Read `AGENTS.md` and the assigned task card first.
- Modify only files explicitly allowed by the task.
- Do not redesign APIs, schemas, or architecture.
- Do not make speculative unrelated cleanup changes.
- Stop and report if the task conflicts with repository reality.
- Run the exact verification commands.
- Add or update tests required by the task.
- Create one focused commit.
- Return a structured handoff with test evidence and questions.
- Never claim success without reporting executed commands and outcomes.

---

## Efficient Context Management

Avoid paying to repeatedly rediscover the repository.

### Keep stable context reusable

Put stable information in repository files or a cache-friendly prompt prefix:

- Build and test commands
- Project architecture summary
- Coding conventions
- Package manager and runtime versions
- Important directory ownership rules
- Authentication and authorization conventions
- Database migration conventions
- Existing patterns for errors, logging, and tests
- Formatting and linting rules

### Send Luna only task-relevant context

A worker generally needs:

- The task card
- Relevant interface/type definitions
- Relevant existing implementation examples
- Relevant test files
- The affected module’s local conventions
- Exact verification commands

A worker generally does **not** need:

- Full feature discussion history
- Every unrelated architecture document
- The entire repository tree
- Every other worker’s raw transcript
- Sol’s private reasoning process
- Open-ended permission to inspect and modify any file

> Prefer durable artifacts, concise task cards, narrow code excerpts, and structured handoffs over long conversational context.

---

## Recommended Default Policy

```md
# Default Sol + Luna Policy

## Sol Owns

- Requirements interpretation
- Architecture and domain decisions
- Task decomposition
- API and schema contracts
- Shared files
- Security-sensitive decisions
- Integration
- Diff review
- Final acceptance

## Luna Owns

- Bounded implementation tasks
- Scoped test creation
- Mechanical refactors
- Type and lint remediation
- Narrow bug fixes
- Repository reconnaissance
- Focused review passes
- Explicit corrections from Sol

## Delegation Rule

Delegate only tasks with:

- Clear ownership
- Explicit inputs and outputs
- Defined allowed files
- Defined forbidden files
- Deterministic acceptance criteria
- Runnable verification commands
- Limited cross-task dependencies

## Escalation Rule

Return work to Sol if:

- Luna fails twice
- Scope expands beyond the task card
- The contract is ambiguous
- Shared files become necessary
- Security or architecture decisions arise
- Integration fails for non-local reasons
```

---

## Practical Starting Configuration

Start conservatively.

| Setting | Recommended starting point |
|---|---|
| Sol coordinators | 1 |
| Concurrent Luna code workers | 2–3 |
| Concurrent Luna research/review workers | 3–6, if results are independently useful |
| Worker isolation | One Git worktree per code-writing worker |
| Required worker output | Commit SHA, diff summary, tests run, results, uncertainties |
| Merge policy | Sol review before merge |
| Final gate | Integration tests, typecheck, lint, relevant browser/E2E tests |
| Retry policy | At most two Luna attempts before Sol escalation |
| Shared files | Sol-owned unless explicitly assigned to one worker |
| Feature planning | Sol-only |

Do not maximize parallelism immediately. Increase worker count only after you observe:

- Low merge-conflict rates
- Stable and useful task-card templates
- Good test coverage
- Reliable worker handoffs
- Few repeated Sol review failures
- Clear interface ownership
- Fast enough CI and local feedback loops

---

## Bottom Line

> **Use Sol for judgment; use Luna for execution.**

Sol should decide **what** to build, **how the system should fit together**, **how to partition work**, and **whether the result is acceptable**.

Luna should implement tightly scoped changes, run deterministic checks, and provide evidence for Sol’s review.

The workflow becomes economical when:

- Sol creates clear contracts.
- Luna receives narrow, testable tasks.
- Workers operate in isolated worktrees.
- Tests and acceptance criteria define “done.”
- Sol reviews diffs and integrates changes.
- The system escalates ambiguity and cross-cutting problems rather than repeatedly retrying cheap workers.

For well-specified web application work, this pattern should normally deliver **substantially more useful code per quota budget** than using Sol alone, while retaining Sol’s higher-level engineering judgment at the points where it matters most.
```
