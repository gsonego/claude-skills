---
name: ship
description: Close out the work on the current branch - commit, push, PR and, if asked, merge and return to the main branch
argument-hint: "[merge] [PR title]"
allowed-tools: Bash, PowerShell, Read, Grep, Glob, Edit, Write
---

Close out the work on the current branch following the flow below.

Arguments received: `$ARGUMENTS`

- If the arguments contain `merge` (or `mesclar`), step 5 is authorized.
  **Without that word, stop after step 4** and just report the PR link.
- The rest of the arguments, if any, is the desired PR title.

## 0. Adapt to the repository

This skill is global and runs in different projects. Before acting, find out:

- **Main branch:** `git symbolic-ref refs/remotes/origin/HEAD` (or `git remote show origin`).
  Do not assume `main`: it may be `master`, `develop`, `trunk`.
- **Language:** write the commit and the PR in the **project's language**: follow what
  `CLAUDE.md`/`AGENTS.md` say, otherwise the predominant language of the commit history.
  With no project signal, use the language the user writes in this session; with no
  signal at all, English.
  **Mandatory exception:** the keyword that closes a GitHub issue (step 4) is syntax,
  not prose. Never translate it.
- **Message convention:** does the history use Conventional Commits (`feat:`, `fix:`)?
  Follow what already exists instead of imposing a new standard.
- **What happens on entering the main branch:** read `.github/workflows/` (or
  `.gitlab-ci.yml`, etc.). If there is automatic deploy, **merging is publishing**, which
  changes the weight of step 5.
- **Repository instructions:** if there is a `CLAUDE.md`/`AGENTS.md`, follow what it says
  about commits, branches and deploy. It takes precedence over this file.

## 1. Get your bearings

- `git branch --show-current`, `git status --short`, `git log --oneline -5`
- If you are **on the main branch**, create a new branch from it before going on. Never
  commit straight to it, and never stop to ask just because of this. Name it from what the
  diff shows (`feat/`/`fix/`/`chore/` prefix by the type of change + a short slug),
  following the naming pattern already used in `git branch -a`/`git log --all --oneline`
  when one is visible. Example: `git checkout -b feat/subject-name`.
- If the tree is already clean and the branch already published, **skip steps 2 and 3**:
  the commit was probably already made. Never create an empty commit just to have
  something to ship.

## 2. Review before describing

Read the `git diff` (or `git show` of what is already committed) **before** writing any
message. The description must reflect what the code actually does.

Also check that no junk got into the diff: screenshots, temp files, build output,
`node_modules`, keys or credentials. If you find any, warn the user instead of
committing on top of it.

## 3. Commit

In the imperative mood, explaining the **why** as well as the what. End with the
attribution trailer your session instructions specify for commits; if they specify none,
add none.

## 4. Push and PR

- `git push -u origin <branch>`
- Check that a PR does not already exist for the branch: `gh pr list --head <branch> --state all`
- **If the work resolves an issue, the PR body must close it automatically.**
  GitHub only recognizes an **English** keyword followed by `#<n>`:
  `Closes #66`, `Fixes #66`, `Resolves #66`. Writing it in another language is plain
  text: the issue stays open and someone finds out later.
  Put `Closes #<n>` on its own line, even if the rest of the body is in another language.
  It only works for issues in the **same** repository; for another repo use
  `Closes owner/repo#n`.
- `gh pr create --base <main-branch>`, with a body covering: context (the problem that
  motivated it), changes, **how it was verified** and what is still pending. End with the
  PR attribution line your session instructions specify, if any.
- If `gh` is unavailable, there is no GitHub remote, or no authentication, **say so
  clearly** and stop after the push. Do not pretend the PR was created.
- Report the link to the user.

## 5. Merge: only if authorized in the arguments

1. **Wait for CI to finish** (`gh pr checks <n>`). If the repository has no CI, tell the
   user: no checks is not a green light, it is the absence of a safety net.
2. **Red check = no merge.** Report the failure and stop. Do not work around it with
   `--admin` or by disabling a check.
3. Green: `gh pr merge <n> --merge --delete-branch` (use `--squash` if that is the visible
   convention in the project history).
4. Go back to the updated main branch: `git checkout <main> && git pull`
5. **Confirm the linked issue closed** (`gh issue view <n> --json state`). If it is still
   open, the step 4 keyword failed: close it by hand
   (`gh issue close <n> --comment "Resolved by PR #<n>"`) and tell the user the
   auto-close did not work.
6. If step 0 indicated automatic deploy, follow it (`gh run list --limit 1`) and report
   the result. Do not declare "published" before confirming.

## 6. Wrap up

Report directly: what was committed, the PR link, whether it was merged, the deploy
state and which branch the user is on now.

If anything was left along the way (a failing check, a pending publish, a step you did
not run, a file you left uncommitted), **say so explicitly**. A partial delivery reported
as complete is worse than a partial delivery.
