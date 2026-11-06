---
name: talvedo
description: How to work with the Talvedo issue tracker through the Talvedo MCP server. Use it for any Talvedo task: reading, filing or updating issues, epics, sprints, releases, tracks, labels, checklists, process diagrams and test runs, especially bulk work such as a batch of issues, sprint planning or splitting a feature across platforms.
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
- **Process** - a diagram of a complex product flow in a project: booking a session, a payment through a bank, a payout. Every edit is a revision people can compare and restore; yours are marked as made by an agent and carry your note.
  - **Lane** - who acts, top to bottom: Client, Therapist, Our server, Monobank, Timer.
  - **Block** - one step in a lane and a column. Kinds: `start`, `step`, `decision` (a fork), `wait` (a webhook, a bank answer, a timer), `end`. Mode: `manual` - a person does it by hand, `auto` - the system does it. `system` names the external system the step works with (Monobank, Google Sheets): that marks an integration; empty - an internal step.
  - **Status of a block** comes from its issues: an open bug - broken; all closed and one done - implemented; open issues - planned. A block without issues has a manual status: potential (default), planned or implemented.
  - **Link** - `next`, `failure` (error, refusal) or `timeout` (time is up), with a condition label ("paid", "no answer in 15 min").
  - **Checks** - dead ends, decisions with one branch, waits without a timeout, external calls without a failure branch, blocks no start reaches. They are hints: fix them or leave them on purpose.
  - **Review** - a person reviews every process and approves a revision of it in the app; agents never approve. `review` is `approved`, `not approved yet` or `changed since approval`: a change of steps, links or lanes after approval waits for a person again, while moving blocks, step statuses and issues do not. The `reviewer` is the person asked to approve it.

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

**Map a process from issues.** `list_processes` first: extend an existing process instead of making a twin. Read the epic and issues of the flow (`search_issues` with `epic` or `text`, `get_issue` for the logic). Name the lanes after who acts. Walk the happy path from a start to an end, then for every wait and every call to an external system add the `failure` and `timeout` branches - this is where the bugs hide. Attach to each block the issues that implement it (`issues` by key); a step nobody has filed stays without issues and shows as potential - list those to the person, file issues only when they ask. Leave `col` and `row` out: blocks line up along the links, a branch into a taken cell goes a row lower in its lane, and independent scenarios of one process (each with its own start) stand side by side instead of mixing into one line. `create_process` in one call, then fix what the checks report with `update_process` and a note on what and why. Pass `reviewer` - the person who asked for the process - so it lands in their "To review" list: only a person approves a process.

**Change a process.** `get_issue` lists the processes and steps an issue stands in (`processes`). `get_process`, then one `update_process` with all operations and a note. Once a person has approved a process, `update_process` does not change it at once: it files a proposal that a person accepts or rejects on the canvas; an outdated proposal (the process changed meanwhile) is rejected - read the process again and propose anew. Do not rebuild a process from scratch to change it: people's edits and the history stay only if you change what needs changing. People also arrange a process on a free canvas: `frames` in `get_process` name its scenarios with the blocks inside and `notes` carry their remarks - read them, they keep their places when you edit.

**Build on a process.** Before you implement steps of a process, check `review` in `get_issue` (`processes`) or `list_processes`. `stepsChangedSinceApproval` in `get_issue` names the steps of your issue that changed after approval and what changed in them. Approved - build what it says. Not approved yet - it is a draft: tell the person. Changed since approval - `get_process` lists `changesSinceApproval`: these steps and links are not reviewed yet, and `get_process` with `revision` = `review.approved.revision` shows the approved version. `list_processes` with `review: pending` finds what waits for a person, and `set_process_reviewer` asks someone to look after you changed a process. `list_inbox` tells the token owner when a process they created, approved, review or work in was approved or changed after approval, with the steps touched: re-read those steps before you go on. When `review` is `changes requested`, a person left `changeRequests`: fix each with `update_process`, then `resolve_process_request` with a note on what you did.

**How the app sells.** A project of a team that sells an Atlassian Marketplace app can have it linked: `get_marketplace_app` gives installs, users and rating with the change over 7 and 30 days, and for the project lead and admins the licenses, revenue over 30 days, the latest sales and uninstall feedback. Read the feedback before you suggest what to build next; quote numbers as they are, in US dollars.

**Testing.** Cases WEB-T12: `create_test_case`; a run on a release or sprint: `create_test_run`; `record_test_result` with `createBug: true` files a bug from a failed step.

## Rules

- Read before you overwrite: pass `descriptionVersion` from `get_issue` to `update_issue` so you do not erase someone's edit.
- Canceling needs `cancelReason`. Deleting goes to the trash for 30 days; ask the person before deleting.
- Credentials: values are never returned; never paste secrets into issues or comments.
- An error text says what is wrong and what is allowed: fix the call instead of retrying it unchanged.
- If a tool from this guide is missing, the MCP client keeps an old tool list: ask the person to reconnect the Talvedo server.
