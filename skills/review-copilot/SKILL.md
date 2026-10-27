---
name: review-copilot
description: Go through the GitHub Copilot code review comments on a PR, fix what is valid, reply to and resolve each thread, and close with a summary
argument-hint: "[pr-number-or-link]"
allowed-tools: Bash, PowerShell, Read, Grep, Glob, Edit, Write
---

Process the comments that GitHub Copilot code review left on the pull request, following
the flow below.

Arguments received: `$ARGUMENTS`

- If it is a PR number or URL, use that PR.
- If empty, detect the PR from the current branch.

## 0. Locate the PR and the repository

- `gh repo view --json owner,name --jq '.owner.login, .name'` for owner/repo.
- PR: if `$ARGUMENTS` has something, `gh pr view <arg> --json number,headRefName,url`;
  otherwise, `gh pr view --json number,headRefName,url` (current branch).
- If there is no PR and no argument, stop and say so. Do not guess which PR the user wants.
- If the local branch is **not** the PR's `headRefName`, warn before switching: run
  `git status` first; if there are uncommitted changes, stop and ask instead of silently
  overwriting.

## 1. Fetch the pending Copilot review threads

The REST API does not expose "resolve" for a review thread; that only exists through
GraphQL. Use a GraphQL query to list the threads with `isResolved` and the author of each
comment:

```graphql
query($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      id
      reviewThreads(first: 100) {
        pageInfo { hasNextPage endCursor }
        nodes {
          id
          isResolved
          path
          line
          comments(first: 20) {
            nodes { databaseId body author { login } }
          }
        }
      }
    }
  }
}
```

Run it with `gh api graphql -f query='...' -f owner=<owner> -f repo=<repo> -F number=<number>`.
If `pageInfo.hasNextPage` is `true`, paginate with `after: $endCursor` until exhausted.

Keep the threads where:
- `isResolved` is `false` (threads already resolved, by any author, are not reprocessed;
  running the command again is safe and idempotent).
- the `author.login` of some comment in the thread contains `copilot`, case-insensitive
  (covers variations such as `copilot-pull-request-reviewer[bot]`).

If no thread is left after the filter, say so and stop. Do not produce a fix summary for
a total of zero items.

## 2. For each pending thread

1. Read the `body` of the thread's comment(s) and the real file at `path`/`line` (the
   comment was made on a diff, so the file may have changed since; check that the line it
   points to still matches what the comment describes).
2. Judge whether it is valid:
   - **Valid** → plan and apply the minimum fix needed, following the project's
     conventions (`CLAUDE.md`/`AGENTS.md` take precedence over personal preference). Then
     reply to the comment confirming what was fixed.
   - **False positive / does not apply** → reply to the comment explaining objectively
     why (a technical reason, not a generic evasion), without changing code.
3. Reply in the thread (keeps the conversation on the right comment instead of creating a
   new loose one):
   `gh api repos/<owner>/<repo>/pulls/<number>/comments/<databaseId>/replies -f body="<reply>"`
   using the `databaseId` of the thread's first comment.
4. Resolve the thread via GraphQL:
   ```graphql
   mutation($threadId: ID!) {
     resolveReviewThread(input: { threadId: $threadId }) {
       thread { id isResolved }
     }
   }
   ```
   `gh api graphql -f query='...' -f threadId=<thread-id>`
5. Note the outcome of the item (fixed / false positive / blocked) for the final summary.

If a comment asks for something ambiguous, or that requires an architecture decision you
are not confident making alone, do not force a risky fix. Mark it as blocked, explain the
reason in the reply, **do not resolve the thread** (leave it for the user to decide) and
move on to the next item.

## 3. Commit and push the fixes

- After processing all threads, `git status`/`git diff` to review what changed before
  describing it.
- If any item produced a code change: one commit covering the fixes, in the imperative
  mood, mentioning that they address the automated code review, with the message in the
  style already used in the repo history (language, Conventional Commits or not). End with
  the attribution trailer your session instructions specify for commits; if they specify
  none, add none.
- `git push` (the branch already exists and is tracked, so no `-u`).
- If **no** item produced a code change (only false positives), do not create an empty
  commit: the replies and resolved threads already close the cycle.
- Never `--force`, never skip hooks (`--no-verify`).

## 4. Close with the summary

Report directly:
- Total items flagged by Copilot in this round.
- How many were fixed.
- How many were marked as false positive (and resolved anyway).
- How many stayed blocked/pending, with the reason for each.
- The PR link.

If anything was left along the way (a blocked item, a pending push, a thread that did not
resolve), say so explicitly. Do not report as done what was not.
