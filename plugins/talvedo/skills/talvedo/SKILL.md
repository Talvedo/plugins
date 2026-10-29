---
name: talvedo
description: How to work with the Talvedo issue tracker through the Talvedo MCP server. Use it for any Talvedo task: reading, filing or updating issues, epics, sprints, releases, tracks, labels, checklists and test runs, especially bulk work such as a batch of issues, sprint planning or splitting a feature across platforms.
---

# Working with Talvedo

Talvedo (https://talvedo.com) is the team's issue tracker. You reach it through the Talvedo MCP server and act on behalf of the person who connected it: their rights, their name in the history, notifications go out as if they did it.

## Model

- **Workspace** - the team. Members have a role and a specialization (iOS, Android, Infrastructure...): `list_members`.
- **Project** - a key (WEB) and its own statuses, issue types and tracks: `list_projects`. Scrum projects have sprints, kanban projects flow without them.
- **Issue** - WEB-142. Status, type, priority, assignee, labels, start and due dates, estimate in points, sprint, release, epic, track. Description and comments are Markdown.
- **Subtask** - a step of one issue (`parent`). **Epic** - groups issues, also of several projects (`create_epic`, then the `epic` field). Never create an issue titled "Epic: ...".
- **Label** - shared by the whole workspace: `list_labels`, `create_label`.
- **Track** - a line of work inside a project (Server, Web, iOS, Android, Infra) with its own board, lead and releases. Tracks are set up with `create_track` and `update_track` by the project lead or an admin.
- **Feature and parts** - a feature is an issue without a track; `split_issue` creates one part per track, each with its own status, assignee and release. The feature status follows its parts. A track where the feature is not needed is marked "not needed".
- **Parity matrix** - features as rows, tracks as columns. It is not an object: it is built from split features. Read it with `get_matrix`.
- **Checklist** - a personal pass over issues; ticking an item never changes the issue.

## Before you start

1. `list_projects` - keys, statuses, types and tracks. Use these names exactly; do not invent statuses.
2. `list_members` when you assign: pick people by specialization.
3. `list_epics`, `list_labels` before creating new ones: reuse what exists.

## Recipes

**File a batch of work.** `create_epic` (or reuse one), then `create_issue` per item with `epic`, type, priority and a Markdown description with acceptance criteria. Link order with `link_issues` (`blocks`). Change many at once with `bulk_update_issues` (up to 100).

**Set up tracks.** `create_track` with `defaults: true` adds Server, Web, iOS and Android (Server goes first). Add others by name (`Infra`), give each a `lead`: `split_issue` assigns parts to the track lead.

**One feature on several platforms.** Create the issue without a track (type feature), then `split_issue` with `skip` for the tracks where it is not needed and `assignees` per track if the lead is not the right person. Follow progress with `get_matrix`: `behindBy` shows what a track lags behind.

**Work on an issue.** Move it to the in-progress status when you start, comment what was done with the commit or pull request when you finish, then move it to done. Something left over is a new issue, not a line in a comment.

**Plan a sprint.** `list_sprints`, `create_sprint`, put issues in with `sprint` (number, `current`, `planned`), `start_sprint`. Completing a sprint and releasing a version are decisions a person makes in the app.

**Checklist.** `create_checklist` from keys or from the conditions of `search_issues` (`from`), share it if the team should see it, tick with `check_checklist_item`.

**Testing.** Cases WEB-T12: `create_test_case`; a run on a release or sprint: `create_test_run`; `record_test_result` with `createBug: true` files a bug from a failed step.

## Rules

- Read before you overwrite: pass `descriptionVersion` from `get_issue` to `update_issue` so you do not erase someone's edit.
- Canceling needs `cancelReason`. Deleting goes to the trash for 30 days; ask the person before deleting.
- Credentials: values are never returned; never paste secrets into issues or comments.
- An error text says what is wrong and what is allowed: fix the call instead of retrying it unchanged.
- If a tool from this guide is missing, the MCP client keeps an old tool list: ask the person to reconnect the Talvedo server.
