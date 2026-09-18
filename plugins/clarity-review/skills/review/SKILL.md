---
description: Reviews a GitHub pull request against ClarityPay's baseline correctness and security bar. Use when asked to review a PR, review this, or check this change for bugs.
allowed-tools: Read, Grep, Glob, LS, mcp__github_inline_comment__create_inline_comment, mcp__github_comment__update_claude_comment
---

# ClarityPay PR review

Review the diff for this pull request for correctness and security bugs. Read
surrounding code as needed — do not judge changed lines in isolation.

## Repo-specific overrides

1. If the repository root has a `REVIEW.md`, read it. Its rules override or
   extend everything below for this repo.
2. If the repository root has a `CLAUDE.md`, read it for project conventions.
   Treat a newly introduced violation of it as a Nit.

## Severity

- **Important** — breaks production behavior, corrupts or leaks data, or
  blocks a safe rollback. Fix before merge.
- **Nit** — everything else worth mentioning: style, naming, minor edge
  cases, refactor suggestions.
- **Pre-existing** — a real bug in code the diff touches but did not
  introduce. Report it; don't block on it.

Report at most 5 Nits per review. If you found more, state the count in the
summary instead of listing every one.

## Verification bar

Before reporting a finding, confirm it against the actual code and cite the
file and line. Do not report a behavior claim inferred only from a name,
comment, or docstring.

## Do not report

- Anything CI already enforces: lint, formatting, type errors.
- Generated files, lockfiles, and vendored or third-party code.

## Output

1. Build the full list of findings before posting anything. Do not post
   this list anywhere — it's only for your own bookkeeping.
2. For each finding, call `mcp__github_inline_comment__create_inline_comment`
   with `confirmed: true`, citing the exact file and line. Do not put
   finding details in the summary comment — only inline.
3. After all inline comments are posted, post one summary comment with a
   one-line tally, e.g. "2 Important, 3 Nit."
4. If nothing was found, post only the summary comment, and say so as its
   first line — don't pad with commentary.
5. Skip draft pull requests, and skip pull requests that already have a
   Claude comment for the current commit.
