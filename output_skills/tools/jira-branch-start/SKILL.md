---
name: jira-branch-start
description: Starts work on a Wallapop ticket — finds my active Jira issue, agrees the branch name with me, and creates the branch. Use when asked to start a ticket, create a branch, or begin work on a WPA issue.
---

STARTER_CHARACTER = 🌿

# Start work on a ticket

Creating the branch is the last step, not the first. The name is agreed with the user, never
guessed.

## 1. Find the active ticket

The Atlassian MCP tools are deferred — load with `ToolSearch` (`select:mcp__claude_ai_Atlassian__searchJiraIssuesUsingJql`) before calling.

```
jql: assignee = currentUser() AND status = "in_development" ORDER BY updated DESC
fields: ["summary", "status"]
```

The active-work status is `in_development`. `"In Progress"` is not a status here and returns zero
results — do not fall back to it and conclude there is no ticket.

- **One hit** → show it (`WPA-112606 — summary`) and confirm it's the right one.
- **Several** → let the user pick (AskUserQuestion).
- **None** → say so and ask for the ticket key. If there genuinely isn't a ticket, ask for the whole
  branch name, since the `WPA-XXXXXX_` format won't apply.

## 2. Agree the name

Format: `WPA-XXXXXX_descriptive-name` — ticket key, underscore, then kebab-case.
Example: `WPA-104282_add-leaf-category-id`.

Propose a descriptive part derived from the ticket summary, but **confirm it** before creating
anything. The summary is often longer or vaguer than a good branch name; a short, specific phrase
beats a transliterated title.

## 3. Create it

Check the starting point first — branch off an up-to-date `master`, not off whatever happens to be
checked out:

```bash
git status --short          # stop if there are uncommitted changes; ask what to do with them
git checkout master && git pull
git checkout -b WPA-XXXXXX_descriptive-name
```

In Wallapop repos (`~/projects/wallapop/*`) committing or pushing to `master` is forbidden — the
branch is what makes the work possible, and the PR is the only route to `master`. Each repo's
`docs/github-guidelines.md` carries its own details; read it if the repo has one.
