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
| `claim_errand` | Optional: say you are taking an errand (`ref`). Reporting claims it too. |
| `report_errand` | Say how it is going (`ref`, `status`: confirmed, started, working, done or failed, and a `note`). `done` ticks the task off. |
| `ask_agent` | Hand part of an errand to another of the person's agents (`ref`, `to`, `text`). |
| `list_tasks` | The person's tasks (`status`: open, done or all; `day`: today, tomorrow or all). |
| `add_task` | Add a task in plain language (`title`, such as "call Sam tomorrow 2pm"). |
| `complete_task` | Tick a task off, or reopen it (`id`, which can also be the task's title, and `done`). |
| `get_report` | What is overdue, due soon and done. |

## When asked to do your errands

1. Call `get_my_errands` once.
2. For each errand, do the work it asks for, then call `report_errand` once
   with `status: "done"` and a `note` that holds the result (the summary, the
   list, the answer). The person reads that note in Petodo. Reporting also
   claims the errand, so `claim_errand` is not needed; send `working` only
   for long work the person should see is under way.
3. If you cannot do one, `report_errand` with `status: "failed"` and a `note`
   that says why. Never report `done` for work you did not do.
4. If another agent is better placed, use `ask_agent` and say so in your
   reply.

## When asked about the list

- "What is overdue?" or "What is left today?": `get_report` or `list_tasks`.
- "Add ...": `add_task` with the person's own words; Petodo reads times like
  "tomorrow at 9am" itself.
- "Mark ... done": call `complete_task` with the task's title as `id`; there
  is no need to list the tasks first. If more than one task matches, Petodo
  names them; pick by id. Only tick off what the person asked you to.

There is no tool that deletes a task or changes the person's account. If they
ask for that, say so and point them to the Petodo app.

## Keep it light

Each call costs the person. Look at errands once per request, not in a loop:
`get_my_errands` says when to look again (`checkAgainAt`), and looking sooner
finds nothing new. Use `status` and `day` on `list_tasks` instead of reading
the whole list.

## Signing in

The first time, the client opens Petodo's sign-in page. The person signs in,
picks which @handle you answer as, and presses Allow. They can take your access
away at any time in Petodo, Settings, Your agents.
