# Petodo plugin for Cursor and Grok Bot

[Petodo](https://petodoapp.com/) is a to-do list with a pet on your desktop and
phone. This plugin makes your agent one of your Petodo agents: start a task
with its handle ("@grok find a hiking trail for Saturday") and, when it is due,
the agent picks it up, works it, reports the result and ticks the task off.

It also lets the agent read your list, add tasks in plain language ("call Sam
tomorrow 2pm"), tick them off, and tell you what is overdue.

## What is in it

- `mcp.json`: Petodo's hosted MCP server, `https://app.petodoapp.com/api/mcp`.
  There is nothing to configure: no key, no variables.
- `skills/petodo-errands`: how to work errands and the list with Petodo's
  tools.

## Setup

1. Install the plugin.
2. The first time the agent uses Petodo, a Petodo sign-in page opens. Sign in
   (email, Google, GitHub or Apple), choose which @handle the agent answers as,
   and press Allow.
3. Ask: "Check my Petodo errands and do them."

Sign-in is OAuth 2.1 with PKCE. The agent sees your tasks and the errands
handed to its own handle, never your email address, settings or devices.
Remove its access any time in Petodo, Settings, Your agents.

You need a Petodo account. Agents are part of Petodo Pro; connecting starts a
free week if you have not had one.

## Tools

| Tool | Reads or writes |
|---|---|
| `get_my_errands` | reads |
| `list_tasks` | reads |
| `get_report` | reads |
| `add_task` | adds a task |
| `ask_agent` | adds an errand for another agent |
| `claim_errand` | changes an errand's status |
| `report_errand` | changes an errand's status; "done" ticks the task off |
| `complete_task` | ticks a task off or reopens it |

None deletes anything or touches the account.

## Links

- Help: https://petodoapp.com/petodo/help/#connectors
- Privacy: https://petodoapp.com/petodo/privacy/
- Terms: https://petodoapp.com/petodo/terms/
- Support: hello@petodoapp.com

## License

MIT, see [LICENSE](LICENSE).
