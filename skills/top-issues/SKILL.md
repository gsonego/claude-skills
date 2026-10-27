---
name: top-issues
description: List the 5 most important issues of the current GitHub repository (or, with --quick, the 5 quickest to resolve), with a justification for each.
argument-hint: "[--quick]"
allowed-tools: Bash(gh issue list:*), Bash(gh repo view:*), Bash(gh auth status:*)
---

Analyze the open issues of the GitHub repository tied to the current directory and present a prioritized top 5. Write the output in the language the user writes in.

## 1. Detect the mode

Look at `$ARGUMENTS`:
- If it contains `--quick` or `-q` → mode = **quickest to resolve** (effort).
- Otherwise → mode = **most severe** (severity).

## 2. Check preconditions

Run `gh repo view --json nameWithOwner` in the current directory to confirm there is a valid GitHub remote and that `gh` is authenticated.

If it fails (no remote, `gh` not installed, or not authenticated): explain clearly which step failed and stop. Never pretend the analysis was done.

## 3. Fetch the open issues

Run, in a single call:

```
gh issue list --state open --limit 1000 --json number,title,body,url,labels,createdAt,comments
```

No extra pagination: the intent is to cover every open issue in the repository, even if that makes the call heavier on large repositories.

If the list comes back empty, say there are no open issues in the repository and stop (do not invent items).

## 4. Score and select the top 5

**Default mode (severity)**: judge each issue by its content (title, body, labels), with no fixed point rubric but using judgment about:
- **Security**: vulnerabilities, data exposure, authentication/authorization flaws.
- **Reliability**: broken critical flows, data loss, crashes.
- **Accuracy**: wrong calculations, data or results delivered to the user (for example wrong values, corrupted or inconsistent information).
- **Reach/impact**: how many users or flows the issue affects.

Existing labels (`bug`, `security`, `critical`, etc.) are one more signal, not the only criterion: issues without a label can also make the top 5 if the content justifies it.

**`--quick` mode (effort)**: judge each issue by its text only (without exploring the repository code, which would make the mode slow), estimating apparent complexity: a simple text/config/value change weighs toward "quick"; a change that seems to cross several layers, require refactoring or has ambiguous scope weighs against.

Pick the 5 issues with the highest priority under the active mode's criterion. If there are fewer than 5 open issues in total, list all of them and mention the total (do not pad with fictional items).

## 5. Output format

Numbered list, in priority order, one block per issue:

```
1. **#123 — <title>** (<url>)
   - Problem: <1–2 sentences summarizing the problem>
   - Why it made the top 5: <specific criterion applied: security/reliability/accuracy/impact, or the reason it is quick to resolve>
```

End with a context line: total open issues analyzed and the mode used (severity or `--quick`).
