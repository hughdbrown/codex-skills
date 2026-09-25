---
name: roborev:pull-request-reviewer
description: Pre-review a pull request to anticipate problems and shorten the review cycle
---

# roborev:pull-request-reviewer

Pre-review a pull request to anticipate problems and shorten the review cycle.

## Usage

```
$roborev:pull-request-reviewer [pr_number_or_branch]
```

## IMPORTANT

This skill requires you to **execute bash commands** to inspect the PR diff, run checks, and analyze the changes. The task is not complete until you present a full pre-review report with actionable items.

## Instructions

When the user invokes `$roborev:pull-request-reviewer [pr_number_or_branch]`:

### 1. Determine the review target

If a PR number is provided, fetch its details:

```bash
gh pr view <pr_number> --json number,title,body,headRefName,baseRefName,additions,deletions,changedFiles,files
```

If a branch name is provided (or no argument, defaulting to the current branch):

```bash
git rev-parse --abbrev-ref HEAD
```

Then get the base branch (usually `main`):

```bash
git merge-base main HEAD
```

If the branch has no commits beyond the base, inform the user there is nothing to review.

### 2. Gather the full diff

Get the complete diff of all changes:

```bash
gh pr diff <pr_number>
```

Or if working from a branch:

```bash
git diff $(git merge-base main HEAD)..HEAD
```

Also get the list of changed files:

```bash
git diff --name-only $(git merge-base main HEAD)..HEAD
```

And the commit log:

```bash
git log --oneline $(git merge-base main HEAD)..HEAD
```

### 3. Run automated checks

Run the project's standard checks before analyzing the code:

```bash
go build ./...
go vet ./...
go test ./...
```

If any check fails, note it as a blocking issue in the report. Do not stop the review -- continue analyzing other aspects.

Also check formatting:

```bash
gofmt -l .
```

If there are unformatted files among the changed files, flag them.

### 4. Analyze for common problem areas

Read the changed files and analyze them against these problem categories, derived from historical review patterns on this project. Check each category systematically:

#### Security (High Priority)

- **Prompt injection**: Does the code embed untrusted text (review output, user input, external data) into prompts sent to agentic execution? Look for `Agentic: true` combined with interpolated strings.
- **Auth/authz gaps**: Do new HTTP endpoints have appropriate access controls? Check new handler registrations in `server.go`.
- **Command injection**: Does code pass unsanitized strings to `exec.Command` or shell invocations?
- **Unbounded input**: Do new API endpoints use `http.MaxBytesReader` or otherwise cap request body size?

#### Concurrency (High Priority)

- **Data races**: Are shared fields protected by mutexes? Look for goroutine lifecycle methods (`Start`/`Stop`) that touch shared state. Check for `sync.WaitGroup` usage where `Add` may race with `Wait`.
- **File descriptor races**: Does code write a temp file and immediately exec it? Ensure `Sync` + `Close` before exec to avoid "text file busy" errors.
- **Channel/context misuse**: Are context cancellations propagated correctly? Do goroutines clean up on context cancel?

#### Error Handling (Medium Priority)

- **Unchecked return values**: Are error returns from `io.Close`, `os.Remove`, database operations, and HTTP writes checked? The project uses `golangci-lint` with `errcheck`.
- **Partial failure handling**: In batch operations, does a failure in one item corrupt the state of others? Check loops that combine multiple fallible operations.
- **Missing error context**: Are errors wrapped with `fmt.Errorf("context: %w", err)` to provide stack trace information?

#### Test Quality (Medium Priority)

- **Test coverage for new code**: Do new functions and branches have corresponding tests?
- **Flaky test patterns**: Look for timing-sensitive assertions, shared mutable state between tests, missing `t.Parallel()`, or goroutine leaks (missing cleanup/cancel).
- **Test isolation**: Do tests use `t.TempDir()` for file operations? Do they avoid polluting production paths (`~/.roborev/`)?

#### Scope and Organization (Medium Priority)

- **Scope creep**: Does the PR include unrelated changes (lint fixes, formatting, refactoring) mixed with feature work? Flag files that seem outside the PR's stated purpose.
- **Large files**: Does the PR add to already-large files instead of splitting them? The project has a pattern of splitting files >1000 lines into focused modules.
- **Commit hygiene**: Are commits logically grouped? Does each commit compile and pass tests independently?

#### Go Idioms (Low Priority)

- **`go fmt` compliance**: All files must be formatted.
- **`go vet` compliance**: All files must pass vet.
- **Deprecated patterns**: Look for `reflect.DeepEqual` in tests (prefer `cmp.Diff`), `interface{}` instead of `any`, deprecated stdlib usage.
- **Consistent style**: Does the code follow the existing project conventions? Check naming, error handling patterns, and test structure (table-driven preferred).

#### Dependency and Build (Low Priority)

- **New dependencies**: Are new external dependencies justified? The project prefers stdlib.
- **Version consistency**: Do `go.mod` versions match what the project targets?
- **Build tags**: Are integration tests properly tagged with `//go:build integration`?

### 5. Check the PR description

Verify the PR has:
- A clear **Summary** section explaining the "why" not just the "what"
- A **Test plan** section with specific, checkable items
- References to related issues (Closes #NNN) where applicable

Flag if any of these are missing or insufficient.

### 6. Generate the pre-review report

Present findings in this format:

```
## Pre-Review Report: PR #NNN — <title>

### Blocking Issues
(Items that will definitely fail review — fix these first)
- ...

### Likely Review Findings
(Items reviewers will flag — address proactively)
- ...

### Suggestions
(Improvements that would strengthen the PR)
- ...

### Checks
- [ ] `go build ./...` — pass/fail
- [ ] `go vet ./...` — pass/fail
- [ ] `go test ./...` — pass/fail (N tests)
- [ ] `gofmt` — clean/N unformatted files
- [ ] PR description — complete/missing sections

### Summary
<1-2 sentences: overall assessment and recommended next steps>
```

Group findings by the categories above (Security, Concurrency, Error Handling, etc.). Include file paths and line numbers for every finding so the user can navigate directly.

### 7. Offer next steps

Based on the findings:

- If there are blocking issues: "I found N blocking issues. Would you like me to fix them?"
- If there are only suggestions: "The PR looks ready for review. Consider addressing the N suggestions to preempt reviewer feedback."
- If clean: "The PR looks clean. No issues anticipated."

If the user wants fixes applied, proceed to fix them, run the test suite, and ask if they want to commit.

## Examples

**Review current branch:**

User: `$roborev:pull-request-reviewer`

Agent:
1. Detects current branch `feat/add-kilo-agent`
2. Computes diff against `main`, finds 4 changed files (+180/-20)
3. Runs `go build`, `go vet`, `go test` -- all pass
4. Analyzes changes:
   - New agent implementation follows existing pattern -- good
   - Missing test for error path when kilo binary not found -- medium
   - New agent not added to shell completion registration -- low
5. Checks PR description -- has Summary and Test Plan
6. Presents report with 2 findings (1 medium, 1 low)
7. Offers: "Would you like me to add the missing error-path test and the completion registration?"

**Review a specific PR:**

User: `$roborev:pull-request-reviewer 278`

Agent:
1. Fetches PR #278 details via `gh pr view`
2. Gets the full diff (40 files, +2550/-195)
3. Runs checks -- all pass
4. Flags 3 blocking issues:
   - Prompt injection: prior review output embedded in agentic prompt (`compact.go:445`)
   - Scope creep: 15 files with unrelated lint fixes
   - Partial failure: `markCompactSourceJobs` can lose data on partial failure
5. Flags 2 likely findings:
   - Unbounded `--limit` can exceed batch API cap
   - Inconsistent validation between CLI and worker paths
6. Presents structured report
7. Offers: "I found 3 blocking issues that will fail review. Would you like me to fix them?"

## See also

- `$roborev:review` — request a code review for a commit (post-commit review by AI agents)
- `$roborev:fix` — fix findings from completed reviews
- `$roborev:address` — address a single review's findings
