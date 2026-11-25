# personal-agent architecture

personal-agent is a derivative of [ZeroClaw](https://github.com/zeroclaw-labs/zeroclaw). It is not affiliated with ZeroClaw Labs. This note maps a file-memory personal agent (inbox, timers, memory, permissions, outbox) onto the runtime that is already in this tree. It does not replace that runtime.

The binary and crate names are still `zeroclaw`. The default Cargo features keep the agent loop, memory, and cron, and they compile email, webhook, filesystem, and the editor ACP path. See the root `README.md` for the feature list.

## What already runs

A configured daemon (`zeroclaw daemon` / `zeroclaw service`) is the always-on process. Channels and the cron scheduler deliver work into the same agent loop. That is a long-running process, not a separate harness that exits after every short run. This release does not add that outer launcher. Timers still do not call a model by themselves: a cron job of type `agent` starts one agent turn when it fires, and a channel message starts one turn when it arrives.

| Primitive | Use the existing surface | This repo's convention |
| --- | --- | --- |
| Inbox | Email (`[channels.email.<alias>]`), webhook (`[channels.webhook.<alias>]`, its own HTTP listener), filesystem watch (`[channels.filesystem.<alias>]`), and ACP editor sessions | `templates/personal-agent-workspace/` watches `workspace/inbox/` for `*.json` when you enable the filesystem channel |
| Timers | `[cron.<alias>]` with `job_type = "agent"`. `schedule.kind = "cron"` repeats; `schedule.kind = "at"` is a one-shot RFC 3339 time. The agent can also create jobs with the cron tools | Daily brief and weekly dreaming samples are in `config.example.toml`, both `enabled = false` until a provider is set |
| Memory | Markdown backend: `MEMORY.md` plus `memory/YYYY-MM-DD.md` under the agent workspace. Every turn also loads `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, and `MEMORY.md` when those files exist (each is truncated; see the upstream troubleshooting note on bootstrap caps). SQLite remains available and is still the upstream default if you do not select markdown | `AGENTS.md` is the always-loaded behavior file. `memory/index.md`, `memory/notes/`, `memory/progress/`, and `memory/reported.md` are ordinary files the agent reads and writes on demand. They are not a new memory engine |
| Permissions | `[risk_profiles.<alias>]` (`readonly`, `supervised`, `full`). `supervised` is the upstream default: low-risk tools run, medium-risk tools ask, high-risk tools are blocked. ACP asks with `session/request_permission`. `always_ask`, `excluded_tools`, and `approval_route` narrow that further | The sample profile is `supervised`. `workspace/permissions/policy.md` tells the agent which actions must wait for approval. Credentials stay in the ZeroClaw config and secret store, not in the workspace |
| Outbox | Cron `delivery.mode = "announce"` sends to a channel (`channel = "email.<alias>"` or `webhook.<alias>`). `mode = "none"` sends nothing. Webhook `send_url` posts replies. Email sends only when SMTP is configured | Sample jobs use `mode = "none"` (quiet). `workspace/outbox/` is a place to write a message you decided not to deliver yet. The dispatcher is still the channel send path, not a new process |

## Memory files

ZeroClaw's markdown memory store is append-oriented. `MEMORY.md` is evergreen core memory. Files named `memory/YYYY-MM-DD.md` are daily logs. Headings are skipped; other non-empty lines are entries. Subdirectories such as `memory/notes/` are **not** scanned by that store. The agent reaches them with the file tools, which is what we want for notes that should load only when the index says they matter.

Put rules that must hold on every run in `AGENTS.md` (the file the runtime already loads). A stable preference that must also be recallable as memory can be mirrored into `MEMORY.md`. `memory/reported.md` is the "already told the user" ledger; the sample `AGENTS.md` says not to repeat it unless the facts changed.

Dreaming is a cron job, not a resident process. The sample `weekly_dreaming` job asks the agent to read recent notes and daily logs, update `memory/notes/` and `memory/index.md`, and not send a message (`delivery.mode = "none"`).

## Workspace template

[`templates/personal-agent-workspace/`](templates/personal-agent-workspace/) is a copy-and-edit starting point:

- `config.example.toml` selects markdown memory, a supervised risk profile, a local gateway for ACP, and commented email / webhook / filesystem / cron blocks
- `workspace/AGENTS.md` is the behavior file loaded every turn
- `workspace/MEMORY.md` and `workspace/memory/` are the file-memory layout

Point `[agents.default.workspace].path` at the `workspace` directory after you copy the template. Leave channel `enabled = false` until the endpoints and secrets are real. Do not commit secrets.

## What this release does not change

Channel and tool source that is off by default is still compiled only when its feature is on; the files are not deleted. Postgres memory, Prometheus, WASM plugins, sandboxes, browser automation, WebAuthn, hardware, and the coding-CLI tools are unchanged and opt-in. There is no new inbox process, no new scheduler, and no rename of the `zeroclaw` binary.
