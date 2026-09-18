---
description: Reviews a GitHub pull request against ClarityPay's baseline correctness and security bar. Use when asked to review a PR, review this, or check this change for bugs.
allowed-tools: Read, Grep, Glob, LS, mcp__github_inline_comment__create_inline_comment, mcp__github_comment__update_claude_comment, Bash(gh api graphql:*), Bash(gh pr view:*), Bash(gh repo view:*)
---

# ClarityPay PR review

## Step 0: unresolved-thread gate (hard check, run first)

Before reading any code, check whether every existing review comment thread
on this pull request is resolved. Run:

```
gh repo view --json owner,name -q '.owner.login + " " + .name'
gh pr view --json number -q '.number'
```

Then query thread status (substitute the owner, name, and number from
above):

```
gh api graphql -f query='
{
  repository(owner: "OWNER", name: "NAME") {
    pullRequest(number: NUMBER) {
      reviewThreads(first: 100) {
        nodes {
          isResolved
          comments(first: 1) { nodes { path body } }
        }
      }
    }
  }
}'
```

- If every thread has `isResolved: true` (or there are no threads), proceed
  to Step 1.
- If any thread has `isResolved: false`, **stop**. Do not read the diff, do
  not call the inline-comment tool. Post one summary comment naming each
  unresolved thread (`path`, and enough of `body` to identify it) and saying
  review is blocked until they're resolved. This is a hard gate — an
  unresolved thread blocks the entire review, not just the finding it
  represents.

## Step 1: review

Review the diff for this pull request for correctness and security bugs.
Read surrounding code as needed — do not judge changed lines in isolation.

### Repo-specific overrides

1. If the repository root has a `REVIEW.md`, read it. Its rules override or
   extend everything below for this repo.
2. If the repository root has a `CLAUDE.md`, read it for project conventions.
   Treat a newly introduced violation of it as a Nit.

### Severity

- **Important** — breaks production behavior, corrupts or leaks data, or
  blocks a safe rollback. Fix before merge.
- **Nit** — everything else worth mentioning: style, naming, minor edge
  cases, refactor suggestions.
- **Pre-existing** — a real bug in code the diff touches but did not
  introduce. Report it; don't block on it.

Report at most 5 Nits per review. If you found more, state the count in the
summary instead of listing every one.

### Verification bar

Before reporting a finding, confirm it against the actual code and cite the
file and line. Do not report a behavior claim inferred only from a name,
comment, or docstring.

### Do not report

- Anything CI already enforces: lint, formatting, type errors.
- Generated files, lockfiles, and vendored or third-party code.

## Step 2: output

1. Build the full list of findings before posting anything. Do not post
   this list anywhere — it's only for your own bookkeeping.
2. For each finding, call `mcp__github_inline_comment__create_inline_comment`
   with `confirmed: true`, citing the exact file and line. Every inline
   comment must follow this exact structure, in order, and nothing else:

   ```
   **[Important|Nit|Pre-existing] <one-line description of the bug>**

   <what's wrong, citing the specific code>

   **Impact:** <the concrete failure this causes — what breaks, what data
   is affected, under what conditions>

   **Fix:** <a specific, actionable fix — not "consider handling this
   better">
   ```

   Skip a section only when it does not apply (e.g. a Nit with no
   meaningful "Impact" beyond readability can omit that line, but never
   omit "Fix"). Be crisp: no hedging, no restating the diff, no filler
   sentences.
3. Do not put finding details in the summary comment — only inline.
4. After all inline comments are posted, post one summary comment with a
   one-line tally, e.g. "2 Important, 3 Nit."
5. If nothing was found, post only the summary comment, and say so as its
   first line — don't pad with commentary.
6. Skip draft pull requests, and skip pull requests that already have a
   Claude comment for the current commit.
