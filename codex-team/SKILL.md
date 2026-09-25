---
name: codex-team
description: Coordinate a Sol technical lead and Luna workers for coding tasks that benefit from bounded delegation. Use when the user requests a Sol/Luna team, cost-conscious agent collaboration, or parallel implementation with central review and integration.
---

# Sol + Luna Codex Team

Use Sol for requirements, architecture, contracts, task decomposition, review, integration, and final acceptance. Use Luna for bounded implementation, reconnaissance, focused verification, and localized repairs. Optimize for accepted work after review and rework, rather than worker count or raw output.

## Establish the team

Inspect the available agent tools, model choices, concurrency limits, repository instructions, and working-tree state before dispatching work. This skill requests delegation when applied to an implementation task; a request to edit or explain this skill does not launch a team.

- Select the available Sol model for the technical lead and Luna for workers. In environments exposing `gpt-6-sol` and `gpt-6-luna`, use those exact identifiers. Explicit user model choices take precedence. Do not silently substitute another model or claim that a role prompt changes the underlying model.
- If the current agent is Sol, coordinate directly. Otherwise, delegate technical leadership to one Sol agent and retain only user communication and orchestration duties. Account for both agents when allocating concurrency; avoid duplicate planning or review.
- Use the runtime's actual model-selection mechanism. With `collaboration.spawn_agent`, select `model` explicitly and use `fork_turns: "none"` for a concise, self-contained assignment; a full-history fork may prohibit model overrides. Follow the current tool schema rather than assuming this interface exists everywhere.
- Start with one lead and up to two Luna workers, constrained by available slots and independent work. Reserve capacity for the lead; worker agents should return questions to it rather than recursively spawning more workers.
- If model selection or delegation is unavailable, disclose the limitation and proceed with useful single-agent work within the user's scope. Do not imply a mixed-model team ran.

Keep very small or tightly coupled changes with Sol when task preparation and review would cost more than direct completion. Do not promise fixed cost or throughput multipliers. If evaluating efficiency, include planning, context transfer, tool execution, review, integration, and retries; report measured usage only when available.

## Assign work by judgment required

| Sol owns | Luna can execute after boundaries are set |
|---|---|
| Product interpretation, scope, acceptance criteria | Targeted repository searches with file and line evidence |
| Architecture, domain models, API and schema contracts | Components, endpoints, persistence changes following a fixed contract |
| Authorization, tenancy, privacy, billing, destructive-data decisions | Specified validation and negative tests under Sol review |
| Shared files, migration sequence, compatibility decisions | Mechanical refactors, generated clients, fixtures, documentation |
| Cross-module debugging and integration | Local bug fixes, lint/type repairs, focused test runs |
| Diff review and final acceptance | Explicit corrections from review |

Security-sensitive decisions stay with Sol. Delegate their implementation only after the policy and failure behavior are explicit and reviewable. Luna must escalate newly discovered policy questions instead of inventing answers.

## Prepare bounded tasks

Read applicable `AGENTS.md` files and inspect relevant implementation patterns and real build/test commands. Delegate narrow reconnaissance when it can answer a concrete question. Require findings with locations and uncertainty, not an ungrounded repository summary.

Sol establishes user-visible behavior, non-goals, acceptance criteria, contracts, and dependencies before implementation begins. Reuse existing planning artifacts. For sustained work, keep concise task records and status in the project's established location; do not create a new documentation hierarchy or rewrite `AGENTS.md` merely to follow this workflow.

Every worker assignment must provide:

```text
Task ID and goal:
Role: Luna worker; no further delegation
Workspace: absolute path; branch and base commit when using Git
Dependencies: accepted prerequisite tasks and contract version/location
Read first: applicable repository instructions, relevant examples and contracts
Allowed files: explicit paths or narrowly scoped globs
Forbidden changes: shared files and behavior outside this assignment
Contract: inputs, outputs, error behavior, invariants, compatibility constraints
Acceptance: observable cases, including relevant failure cases
Verification: exact commands, working directory, required environment
Deliverable: diff or focused commit, verification evidence, structured handoff
Escalation: stop on ambiguity, boundary expansion, or design changes
```

Supply task-relevant context, not the entire conversation or other workers' transcripts. Workers may inspect related code as needed; write ownership remains bounded. Read-only assignments should explicitly prohibit edits and return evidence instead of a commit.

For example, replace “implement organization invitations” with a Sol-defined contract and separate tasks for persistence, repository methods, endpoints, client bindings, and UI. Specify administrator access, organization scoping, expiration behavior, and token secrecy centrally. An endpoint task is ready only when its data-access contract is stable. Two endpoints that modify the same service file are not independent tasks.

## Isolate and schedule implementation

Prefer separate Git worktrees for code-writing workers. A separate branch in the same checkout does not isolate concurrent edits. Sol prepares each worktree from a known integration base and provides its absolute path; workers must run tools in that path rather than assume their default directory changed.

- Preserve existing user changes. Do not reset, stash, or overwrite them to prepare worker environments.
- Incorporate accepted dependencies into the worker's base before starting dependent implementation. If uncommitted changes are required, arrange an explicit, reviewable transfer; do not let workers operate on a stale approximation.
- Give each mutable file one owner at a time, including shared schemas, routing, configuration, lockfiles, and generated output. Assign shared changes to Sol or serialize them.
- Worktrees do not isolate databases, ports, external services, or migration numbering. Allocate independent test resources or serialize those operations.
- If worktrees are unavailable or the task is outside Git, use bounded diffs and serialize writes unless disjoint ownership and independent verification are clear. Do not initialize a repository solely for delegation.
- Parallelize independent investigation freely within useful capacity. Parallelize edits only when contracts are stable and files and mutable test resources do not overlap.

Track task status, owner, workspace, dependencies, attempt count, and evidence. A useful sequence is `ready → running → review → accepted`, with `blocked` or `needs-repair` as needed. Worker completion means ready for review, not accepted.

While workers execute, Sol handles contracts, review, and integration that do not duplicate worker assignments. Use the runtime's messaging and follow-up tools to resolve questions and reuse relevant worker context. If a contract changes, pause affected work and update assignments before continuing.

## Require evidence from Luna

Workers read their task card and applicable repository instructions, implement only the assigned scope, and execute the specified checks. They must not weaken tests or change acceptance criteria to produce a passing result. If a command is invalid or an environment prerequisite is missing, report the actual failure and proposed equivalent verification.

Use one focused commit per completed task when the repository workflow permits it; otherwise return an inspectable diff. Workers must not merge into the integration branch, publish changes, or deploy as part of a task handoff.

Require this concise handoff:

```text
Task / status: ready-for-review, needs-repair, or blocked
Summary: behavior implemented or findings established
Location: workspace, branch, base and commit SHA; or explicit diff location
Files changed:
Acceptance evidence: which cases were checked and how
Verification: commands, working directory, outcomes, relevant failure output
Tests added or changed:
Uncertainties: limitations, unrun checks, blockers, decisions needed
```

Distinguish checks that passed, failed, and were not run. “Implemented successfully” and a commit SHA alone are insufficient evidence.

## Review, repair, and integrate

Sol inspects the actual diff and relevant test evidence for every Luna-authored change before acceptance. Check scope and contract compliance, repository conventions, error behavior, meaningful tests, and applicable security, tenancy, compatibility, or data-loss concerns. Passing CI supports review; it does not replace it.

Send localized defects back with the failing case, expected behavior, allowed files, and verification command. Allow at most two Luna implementation attempts per task: the initial attempt and one focused repair. If the second attempt still fails acceptance, Sol diagnoses and takes over or materially respecifies the task. Do not reset the attempt count by renaming the same failing task.

Escalate immediately to Sol when a task requires writes outside its boundary, has an ambiguous contract, exposes an architectural flaw, conflicts with another task, or needs a security/product decision. Luna reports the evidence and stops affected work. Sol clarifies the contract, serializes work, narrows the assignment, or implements the difficult part. Ask the user only for decisions that cannot be resolved from the authorized scope and available evidence.

Integrate accepted commits into the designated local integration checkout in dependency order. For changes already made in a shared checkout, review them in place; do not cherry-pick the same changes again. Resolve conflicts centrally, and rerun checks affected by conflict resolution or changed dependencies.

Run the repository's required checks plus relevant integration and end-to-end verification on the assembled result. Avoid redundant full-suite runs for every isolated worker; batch integration checks when appropriate, while retaining focused worker evidence. A worker's passing tests do not establish that the combined feature works.

Mark the overall task complete only when user acceptance criteria and required checks pass. If verification remains blocked, report precisely what is implemented and what is unverified. Final reporting should summarize the result, validation, and remaining limitations. Local integration does not grant permission to push, merge a remote PR, release, or deploy beyond the user's existing authorization.

## Basis and maintenance

This workflow condenses the supplied `PROMPT.md`. The model identifiers above reflect a runtime that exposes those choices; inspect the active environment before using them. For general delegation concepts, see [OpenAI's multi-agent guidance](https://developers.openai.com/api/docs/guides/agents-api/multi-agent). Runtime tool definitions remain authoritative for model overrides, context inheritance, and concurrency.
