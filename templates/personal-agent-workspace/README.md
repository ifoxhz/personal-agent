# personal-agent workspace template

Copy this directory somewhere persistent (the memory has to outlive a single process). Then:

1. Copy `config.example.toml` to `~/.zeroclaw/config.toml`, or pass that file to the `zeroclaw` binary with the config flag your install already documents.
2. Set `[agents.default.workspace].path` to the absolute path of the `workspace/` directory in the copy.
3. Point `[providers.models.ollama.local]` at a model you actually run, or replace that block with another provider from the inherited docs.
4. Enable a channel or a cron job only after its endpoints are real. Samples ship `enabled = false`.
5. Run the default build: `cargo build --release --locked`, then `zeroclaw` from `target/release/`.

This template does not start a second harness. It fills in ZeroClaw's config, markdown memory, cron, and channel fields. See `../../ARCHITECTURE.md`.

Do not put API keys, mailbox passwords, or webhook secrets in this tree. The example uses placeholders.
