---
name: petodo-errands
description: Work the errands a person hands you on their Petodo to-do list, and read or update that list. Use when the person mentions Petodo, their to-do list, their errands, or a task they started with your @handle.
---

# Petodo errands

Petodo is a to-do list. A task that starts with an agent's handle, such as
"@grok find a hiking trail for Saturday", is an errand for that agent once it
is due. The Petodo MCP server gives you these tools:

| Tool | What it does |
|---|---|
| `get_my_errands` | The errands due for you now, each with a `ref`. |
| `claim_errand` | Say you are taking an errand (`ref`). |
| `report_errand` | Say how it is going (`ref`, `status`: confirmed, started, working, done or failed, and a `note`). `done` ticks the task off. |
| `ask_agent` | Hand part of an errand to another of the person's agents (`ref`, `to`, `text`). |
| `list_tasks` | The person's tasks (`status`: open, done or all; `day`: today, tomorrow or all). |
| `add_task` | Add a task in plain language (`title`, such as "call Sam tomorrow 2pm"). |
| `complete_task` | Tick a task off, or reopen it (`id`, `done`). |
| `get_report` | What is overdue, due soon and done. |

## When asked to do your errands

1. Call `get_my_errands`.
2. For each errand: `claim_errand`, do the work it asks for, then
   `report_errand` with `status: "done"` and a `note` that holds the result
   (the summary, the list, the answer). The person reads that note in Petodo.
3. If you cannot do one, `report_errand` with `status: "failed"` and a `note`
   that says why. Never report `done` for work you did not do.
4. If another agent is better placed, use `ask_agent` and say so in your
   reply.

## When asked about the list

- "What is overdue?" or "What is left today?": `get_report` or `list_tasks`.
- "Add ...": `add_task` with the person's own words; Petodo reads times like
  "tomorrow at 9am" itself.
- "Mark ... done": find it with `list_tasks`, then `complete_task`. Only tick
  off what the person asked you to.

There is no tool that deletes a task or changes the person's account. If they
ask for that, say so and point them to the Petodo app.

## Signing in

The first time, the client opens Petodo's sign-in page. The person signs in,
picks which @handle you answer as, and presses Allow. They can take your access
away at any time in Petodo, Settings, Your agents.
