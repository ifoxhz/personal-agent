# personal-agent

personal-agent is a trimmed, file-memory-friendly build of an always-on personal agent runtime.

It is a derivative of [ZeroClaw](https://github.com/zeroclaw-labs/zeroclaw) by ZeroClaw Labs. This repository is **not** the official ZeroClaw project and is **not** affiliated with, endorsed by, or maintained by ZeroClaw Labs. The ZeroClaw name and logo are trademarks of their owner; this project does not use that logo as its brand. Upstream attribution stays in `NOTICE`, `LICENSE-MIT`, and `LICENSE-APACHE`.

The Rust crate names and the command-line binary are still `zeroclaw`. Renaming every crate is intentionally out of scope for this first release. Config still lives at `~/.zeroclaw/config.toml` unless you pass another config directory.

## What a default build includes

`cargo build --release` (and a default `./install.sh` source build) enables:

| Feature | Role |
| --- | --- |
| `agent-runtime` | Agent loop, tools, SQLite and markdown memory, cron and other timers |
| `channel-email` | IMAP / SMTP |
| `channel-webhook` | The webhook channel's own HTTP listener (not the gateway `/webhook` route) |
| `channel-filesystem` | Watches directories and feeds changes into the agent or SOP engine |
| `channel-acp-server` | Editor / Agent Client Protocol server |
| `acp-bridge` | `zeroclaw-acp-bridge`, which dials the gateway ACP WebSocket |
| `gateway` | HTTP gateway, including ACP over WebSocket at `/acp` |

The gateway stays in the default set because the editor bridge talks to it. The webhook *channel* also has its own axum listener and does not require the gateway to receive POSTs.

These stay **off** unless you pass `--features`:

- Chat and social channels: Slack, WhatsApp (including `whatsapp-web`), Matrix, Telegram, Discord, and the rest of `channels-full`
- Hardware: `hardware`, `peripheral-rpi`, `probe`, `dev-sim`
- `tools-coding-cli` and the individual coding-CLI tools (Codex, Claude Code, Gemini CLI, OpenCode, and the Claude Code runner)
- `tools-saas`, `tools-external`, and `browser-native`
- `memory-postgres`
- `observability-prometheus`, `observability-otel`, WASM plugins, `sandbox-landlock`, `sandbox-bubblewrap`, `webauthn`
- `schema-export` (JSON Schema for config). The `zeroclaw-gateway` crate still builds with its own `schema-export` and `a2a` defaults when the gateway dependency is compiled.

Source for the unused channels and tools is still in the tree. Turn a channel back on without deleting the trim, for example:

```sh
cargo build --release --features channel-telegram
cargo build --release --features channels-full
cargo build --release --features observability-prometheus,schema-export
```

`--features` adds to the default set. To build a different set from scratch, pass `--no-default-features` and list what you want. `channels-full` still contains Telegram, Discord, and the other historical channels; they were removed from `default-channels` only.

## Build

Rust 1.96.0 or newer (edition 2024):

```sh
cargo build --release --locked
./target/release/zeroclaw --help
```

The release profile uses fat LTO and one codegen unit, which is slow and memory-hungry. `--profile ci` or `--profile release-fast` are upstream alternatives when a machine cannot finish `--release`.

A workspace you can copy into place is in [`templates/personal-agent-workspace/`](templates/personal-agent-workspace/). How inbox, timers, memory, permissions, and outbox sit on the existing runtime is in [`ARCHITECTURE.md`](ARCHITECTURE.md).

## Upstream docs

The `docs/book/` tree is the inherited ZeroClaw manual. Read it for provider, channel, cron, and security details. Where it still says "ZeroClaw", it is describing that upstream runtime. personal-agent's default channel set is the one in this README, not the wider bundle upstream ships.

## License

Dual-licensed [MIT](LICENSE-MIT) OR [Apache-2.0](LICENSE-APACHE). You may choose either. Copyright and trademark notices are retained; see [NOTICE](NOTICE). Contributors to the upstream project retain copyright in their contributions.
