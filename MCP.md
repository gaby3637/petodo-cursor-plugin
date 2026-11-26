# Petodo MCP server

The documentation for Petodo's hosted MCP server: what it connects to, how
people sign in, and what each tool reads, writes and returns. The plugin in
this repository points at this server; any MCP client can use it directly.

## Endpoint

- Address: `https://app.petodoapp.com/api/mcp`
- Transport: streamable HTTP (JSON-RPC 2.0 over POST, plain JSON answers; no
  server-sent events needed). Protocol versions 2024-11-05, 2025-03-26,
  2025-06-18 and 2025-11-25.
- One address for everyone; it works in production only (there is no
  separate test environment). Reviewers use a dedicated test account.

## Signing in (OAuth 2.1)

- A request with no key gets `401` and
  `WWW-Authenticate: Bearer resource_metadata="https://app.petodoapp.com/.well-known/oauth-protected-resource/api/mcp", scope="agent"`.
- Protected resource metadata (RFC 9728):
  `https://app.petodoapp.com/.well-known/oauth-protected-resource/api/mcp`.
- Authorization server metadata (RFC 8414):
  `https://app.petodoapp.com/.well-known/oauth-authorization-server`.
- Clients: Dynamic Client Registration (RFC 7591, `POST /api/oauth/register`),
  Client ID Metadata Documents, or a client registered once for a platform
  (with a client secret, `client_secret_basic` or `client_secret_post`).
- Authorization code with PKCE, `S256` only. The `resource` parameter is
  checked (RFC 8707) and every authorization answer carries `iss` (RFC 9207).
- Scopes: `agent` (the errands and the list) and `agent:read` (the same,
  read only). The consent page also has an "Only let it read" box. A read-only
  connection is shown only `get_my_errands`, `list_tasks` and `get_report`,
  and every change is refused (`403`).
- The person signs in on Petodo's own page (email link, Google, GitHub or
  Apple; reviewer accounts use a password), sees what the app can and cannot
  do, picks which @handle it answers as, and presses Allow or Cancel.
- The access token is an agent key. It does not expire; it lasts until the
  person removes it (Petodo, Settings, Your agents) or the client revokes it
  (`POST /api/oauth/revoke`, RFC 7009). No refresh tokens.
- An authorization code works once, within five minutes. Using a code twice
  revokes the key it made.

## What a connection can and cannot reach

It can: read the person's tasks, read the errands handed to its own @handle,
report on those errands, ask the person's other agents for help, add tasks,
and tick tasks off or reopen them.

It cannot: delete anything, see the person's email address, settings,
devices, other keys or billing, connect other services, or act for another
@handle.

## Tools

Every tool has a `title` and `readOnlyHint`, `destructiveHint` and
`openWorldHint` as booleans. `openWorldHint` is false for all: they reach only
the person's own list. Results come back as JSON text and as
`structuredContent`; a failure comes back as a tool result with `isError:
true` and a plain sentence saying what went wrong.

| Tool | Class | Sensitive | readOnly | destructive |
|---|---|---|---|---|
| `get_my_errands` | Read | no | true | false |
| `list_tasks` | Read | no | true | false |
| `get_report` | Read | no | true | false |
| `add_task` | Write | no | false | false |
| `ask_agent` | Write | no | false | false |
| `claim_errand` | Write | no | false | true |
| `report_errand` | Write | no | false | true |
| `complete_task` | Write | no | false | true |

None is a sensitive write: nothing sends a message, makes a purchase, moves
money or deletes data. The three marked destructive change a task or errand
that already exists, and each can be undone (an errand can be reported again;
a task can be reopened).

### get_my_errands (Read)

The errands due now for this connection's @handle.

- Input: none.
- Output: `you` (the handles it answers as), `count`, `errands` (each with
  `ref`, a six-character id, plus `to`, `group`, `text`, `dueAt`, `status`),
  `nextDueAt` and `checkAgainAt` (when to look again).
- Side effects: none, apart from noting when the agent last checked in.

### claim_errand (Write)

Say the agent is taking an errand.

- Input: `ref` (required).
- Output: the errand's new state.
- Side effects: the errand's status becomes `confirmed`; the person sees it
  in Petodo.

### report_errand (Write)

Say how an errand is going.

- Input: `ref` (required), `status` (required: `confirmed`, `started`,
  `working`, `done` or `failed`), `note` (the result or the reason), `as` (the
  handle to answer as, when the connection has more than one).
- Output: the errand's new state.
- Side effects: `done` ticks the task off on the person's list and shows the
  note with it; `failed` shows the reason. Reporting again replaces the
  status.

### ask_agent (Write)

Hand part of an errand to another of the person's agents.

- Input: `ref` (required), `to` (the other agent's @handle, required),
  `text` (what to ask, required).
- Output: what Petodo did with the request.
- Side effects: a new task for that handle appears on the person's list.

### list_tasks (Read)

The person's tasks.

- Input: `status` (`open`, `done` or `all`), `day` (`today`, `tomorrow` or
  `all`). Both optional.
- Output: `count` and `tasks` (each with `id`, `title`, `done`, `day`,
  `dueAt`).
- Side effects: none.

### add_task (Write)

Add a task. Times written in plain language ("call Sam tomorrow 2pm") are
read in the person's own time zone.

- Input: `title` (required), `day` (`today` or `tomorrow`), `dueAt` (an ISO
  time).
- Output: `created`, the new task.
- Side effects: the task appears on the person's list and devices.

### complete_task (Write)

Tick a task off, or reopen it.

- Input: `id` (required, from `list_tasks`), `done` (default true; false
  reopens).
- Output: `ok`, `id`, `done`.
- Side effects: the task is ticked off or reopened on every device. For a
  repeating task, today's copy is the one ticked off.

### get_report (Read)

A summary of the list: what is overdue, due soon and done.

- Input: none.
- Output: the summary.
- Side effects: none.

## Errors and limits

| Answer | When |
|---|---|
| `401` with `error="invalid_token"` | The key is wrong or was removed. Sign in again. |
| `402` (JSON-RPC `-32002`) | The account does not include agent access (no Petodo Pro, or its trial ended). The message says only that; the account owner sees why in Petodo. |
| `403` (JSON-RPC `-32001`) | The key may not do this (an account route, or a change on a read-only connection). |
| `404` | No such task or errand (`ref` and `id` come from the read tools). |
| `429` | Too many requests; try again later. |

Rate limits: 120 errand reads an hour per key (polling every 30 seconds
fits), 600 changes an hour per account, 60 authorization requests an hour per
IP address.

## Who can use it

A Petodo account (free; sign in with email, Google, GitHub or Apple).
Agents are part of Petodo Pro: connecting starts a free week of Pro if the
account has not had one; after that Pro is a paid subscription.

## Privacy and support

- Privacy policy: https://petodoapp.com/petodo/privacy/ (see "Connecting
  Petodo from an AI app")
- Terms: https://petodoapp.com/petodo/terms/
- Help: https://petodoapp.com/petodo/help/#connectors
- Support and security contact: hello@petodoapp.com
