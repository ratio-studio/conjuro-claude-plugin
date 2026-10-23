---
name: conjuro-write-gate
description: Decide what from a work session belongs in Conjuro and propose it as one short digest (task updates, completions, new tasks, a Draft project update). Use at the end of a work session, before any Conjuro write the user didn't ask for word for word, or when the user asks what should go into Conjuro.
---

# Conjuro write gate

Conjuro is the source of truth for project work, but it must not become a log of Claude sessions. Only work that a person on the project would track goes in. Reading from Conjuro is never gated; writing always is.

## Which project

Use the project named in the working folder's `CLAUDE.md` under **Conjuro project** (`projectId: …`). If there is none, ask the user once which Conjuro project the session belongs to (offer the active ones from `conjuro_list_projects`) and suggest adding that section to `CLAUDE.md`.

## Gate 1: rules

Collect candidates during the session; don't write as you go. A candidate may only be one of these kinds:

| Kind | Use for |
|---|---|
| `task` | New work with a clear next action and outcome |
| `task_update` | Change to an existing task: done, new due date, blocker, important note |
| `project_update` | A decision, a blocker, or a milestone reached (at most one per project per session) |
| `lead_update` | A real change to a lead or customer |

Never make these into candidates (keep them in project notes or memory instead):

- what Claude did, read, ran, or tested; summaries of the chat
- exploration, intermediate steps, retries, debugging trails
- anything already recorded in git, a PR, or project docs
- secrets, credentials, or personal data that isn't already in Conjuro

Before proposing a new task, search the project's open tasks (`conjuro_search_tasks` with the `projectId`). Prefer a change to an existing task over a new one; a new task that duplicates an open task becomes a `task_update` to it.

Every new task gets a **category**. Call `conjuro_list_categories` with `kind: "task"` once per session and choose by each category's `notes`. Use the `isDefault` one when nothing fits better, and ask when two fit equally. Show the category name in the digest and pass its id as `categoryId`.

**Ask about scheduling; never pick a date yourself.** In the digest, ask once for the batch: today, this week, this month, a specific date, or unscheduled. Then set the matching `timeframe` (`day`, `week`, `month`, or `year`) and a `dueDate` at the end of that period. Unscheduled tasks have no due date, so they appear only in the collapsed "Unscheduled" section of the task view.

## Gate 2: score

Give each candidate a probability that someone on the project would want it in Conjuro, judged against the project's outcome (from `conjuro_list_projects`) and its open tasks.

If the `jev` command is available, use it instead of your own judgment: write the candidates to a JSON file and run `jev gate <file>` with `project`, `description` (one line on what the project is), `outcome`, `existing` (open tasks as `{id, name}`), and `candidates` (`{kind, text, reason, targetId}`). Exit 0 means something is proposed, 1 means nothing passed, 2 means Jev was unavailable (fall back to your own scores).

Sort by score:

- **Propose:** 0.8 and above. At most the 5 best new tasks and the best project update; further ones go to **Held for review** rather than being dropped.
- **Probably not needed:** 0.5 to 0.8.
- **Kept local:** below 0.5, or blocked by the rules.

## Gate 3: the digest

Show the user the digest once per session, as one short list:

1. **Propose** items, each with the exact text and target (project, task, customer), and the category for new tasks.
2. **Held for review** items, with the cap they went over. Suggest merging related ones when that would read better in Conjuro.
3. **Probably not needed** items, one line each, so the user can promote one.
4. A count of items kept local, listed only if the user asks.
5. The one scheduling question for new tasks.

Ask once which to write. Then write only the approved items: `conjuro_update_task` or `conjuro_complete_task` for task updates, `conjuro_create_task` for new tasks, `conjuro_create_project_update` for the project update (always created as a Draft that a person publishes), and the lead tools for lead updates. Give each its own fresh `idempotencyKey` where the tool takes one, and report what Conjuro confirmed. If `jev` scored the candidates, record the decisions with `jev gate --outcome <id>=approved,<id>=rejected`.

If there is nothing to propose, say so in one line ("Nothing for Conjuro from this session") and don't show the digest.

## Exceptions

- When the user explicitly asks for a specific write ("add a task for X", "mark Y done"), do it after the usual confirmation without running the gate.
- Chat replies, meetings, and invoices are never part of the digest. Do them only when asked, with their own confirmation (see the `conjuro-chat`, `conjuro-meetings`, and `conjuro-invoices` skills).
