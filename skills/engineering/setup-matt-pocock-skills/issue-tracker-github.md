# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. Use the GitHub MCP connector for all operations; do not shell out to the `gh` CLI. Tool names below are GitHub MCP functions (`github_app.*`).

## Conventions

- **Create an issue**: `issue_write` (method `create`) with owner, repo, title and body.
- **Read an issue**: `issue_read` (method `get`) for the issue body, labels and comments.
- **List issues**: `list_issues` (or `search_issues` for targeted queries) with the appropriate state and label filters.
- **Make an issue a sub-issue of a parent**: `sub_issue_write`. Without sub-issues, put `Part of #<parent>` at the top of the child body.
- **Blocking**: the GitHub MCP has no tool for native issue dependencies, so use a `Blocked by: #<n>, #<n>` line at the top of the child body (`issue_write`, method `update`). A ticket is unblocked when every blocker is closed.
- **Count open blockers**: read every issue named in the `Blocked by` line with `issue_read` and count those still open.
- **Comment on an issue**: `add_issue_comment` (`update_issue_comment` to edit an existing comment).
- **Apply / remove labels**: `issue_write` (method `update`). GitHub creates a label implicitly the first time it is applied; `get_label` tells you whether it already exists.
- **Close**: post the explanation with `add_issue_comment`, then `issue_write` (method `update`) with state `closed` and a `state_reason`.

Infer owner and repo from `git remote -v` and pass them explicitly to every MCP call; the MCP does not infer the repository from the working directory.

File issues only into the user's own repo. Never file into `mattpocock/skills` (the repo these skills come from) unless the user owns it. In a fork, the default remote may point at the upstream parent, so before the first `issue_write` of a session, confirm the target repo with the user if it is not their repo.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the GitHub MCP PR tools:

- **Read a PR**: `pull_request_read` for the PR and its comments.
- **List external PRs for triage**: `list_pull_requests` (state open), keeping only PRs whose author is not an OWNER, MEMBER or COLLABORATOR of the repo.
- **Comment / label / close**: `add_issue_comment` (works on PRs, which share the issue comment surface), `update_pull_request` for labels and state.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either: resolve it with `pull_request_read` first and fall back to `issue_read`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Read it as in **Read an issue** above.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `issue_write` (method `create`) with the `wayfinder:map` label.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`sub_issue_write`). Where sub-issues are not available, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: see **Blocking** under Conventions.
- **Frontier query**: list the map's open children (`list_issues`, state open, scoped to the map's sub-issues / task list), drop any with an open blocker (see **Count open blockers**, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
- **Claim**: `issue_write` (method `update`) assigning the driving dev, the session's first write.
- **Resolve**: `add_issue_comment` with the answer, then `issue_write` (method `update`) closing the ticket, then append a context pointer (gist + link) to the map's Decisions-so-far.
