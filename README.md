# claude-skills

My personal collection of [Claude Code](https://claude.com/claude-code) skills, packaged as the plugin **`gsn`** (my initials, short to type: `/gsn:ship`). The repo is also a marketplace, so it can list other plugins of mine, such as [chat-checklist](https://github.com/gsonego/claude-chat-checklist).

## Skills

| Skill | What it does |
|---|---|
| `/gsn:clarify [topic]` | Interviews you one question at a time, with selectable answers (recommended one first), until doubts and loose ends about the topic are gone. Complex questions offer an "Explain this first" option. Ends with a summary of decisions, assumed defaults and risks. |
| `/gsn:ship [merge] [PR title]` | Closes out the current branch: review the diff, commit, push and open the PR. With `merge`, waits for CI, merges, returns to the main branch and confirms the linked issue closed. Commit and PR use the project's language. |
| `/gsn:review-copilot [pr]` | Goes through the GitHub Copilot review threads on a PR: fixes what is valid, replies to and resolves each thread, pushes, and summarizes. |
| `/gsn:top-issues [--quick]` | Lists the 5 most severe open issues of the repo (or, with `--quick`, the 5 quickest to resolve), with a justification each. |

## Install

In the terminal, inside Claude Code:

```
/plugin marketplace add gsonego/claude-skills
/plugin install gsn@gsn
```

From a shell:

```
claude plugin marketplace add gsonego/claude-skills
claude plugin install gsn@gsn
```

In the desktop app, where `/plugin` does not run: click **+** next to the prompt box, then **Plugins**, **Add marketplace** with `gsonego/claude-skills`, then **Add plugin**.

## Update

```
claude plugin marketplace update gsn
claude plugin update gsn@gsn
```

Then run `/reload-plugins` in the open session. Third-party marketplaces do not update by themselves; to turn that on, use `/plugin`, then **Marketplaces**, select `gsn` and choose **Enable auto-update**.

## Adding a skill

Create `skills/<name>/SKILL.md` with `name`, `description` and, optionally, `argument-hint` and `allowed-tools` in the frontmatter, bump `version` in `.claude-plugin/plugin.json`, and update this table.

## License

MIT
