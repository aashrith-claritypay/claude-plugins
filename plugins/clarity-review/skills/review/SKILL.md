---
description: Reviews a GitHub pull request against ClarityPay's baseline correctness and security bar. Use when asked to review a PR, review this, or check this change for bugs.
allowed-tools: Read, Grep, Glob, LS, mcp__github_inline_comment__create_inline_comment, mcp__github_comment__update_claude_comment, Bash(gh api graphql:*), Bash(gh pr view:*), Bash(gh repo view:*), Bash(gh pr diff:*)
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

Get the authoritative file list yourself — don't rely only on whatever diff
context you were handed, it can be wrong for newly-added files:

```
gh pr diff --name-only
```

Read every file that command lists, including new files, deploy/build
scripts, and docs — not just the files a pre-built context happened to
surface. Then review the diff for correctness and security bugs. Read
surrounding code as needed — do not judge changed lines in isolation.

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
   **[Important|Nit|Pre-existing] <title, ≤12 words>**

   <1-2 sentences: what's wrong, naming the specific code>

   **Impact:** <1 sentence: the concrete failure — what breaks, under
   what condition>

   **Fix:** <1 sentence, imperative: the specific change to make>
   ```

   Hard caps, no exceptions: title ≤12 words, body ≤2 sentences, Impact
   and Fix ≤1 sentence each. Omit "Impact" only for a Nit with no real
   consequence beyond readability — never omit "Fix".

   Banned: hedging ("might", "could potentially", "worth considering"),
   restating the diff back to the author, explaining what the code does
   before saying what's wrong with it, and citing more than one
   supporting line unless every one is load-bearing to the bug itself.
   State the bug, its consequence, its fix. Nothing else.
3. Do not put finding details in the summary comment — only inline.
4. After all inline comments are posted, post one summary comment with
   exactly this structure:

   ```
   ## Review summary

   **State:** <"Blocking — N Important issue(s), do not merge as-is" if
   any Important findings exist, otherwise "No blocking issues">

   **Tally:** N Important, N Nit, N Pre-existing

   **Scope:** <files actually reviewed, comma-separated> · effort=<level>
   · model=<model> · <"REVIEW.md applied" or "no REVIEW.md in repo">
   ```

   No other prose in the summary comment. Findings live inline; this
   comment is state + scope only.
5. If nothing was found, use the same structure — **State:** "No blocking
   issues", **Tally:** "0 Important, 0 Nit, 0 Pre-existing" — don't pad
   with extra commentary.
6. Skip draft pull requests, and skip pull requests that already have a
   Claude comment for the current commit.
