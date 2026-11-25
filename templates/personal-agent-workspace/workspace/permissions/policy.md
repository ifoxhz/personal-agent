# Approval policy

The risk profile in `config.example.toml` is `supervised`. Low-risk reads run. Medium-risk writes and shell commands ask. High-risk actions are blocked.

Always wait for a person before you:

- message anyone other than the user
- create, change, or cancel a calendar event or a task in an external system
- spend money or start a subscription
- delete or bulk-rewrite data outside this workspace

Reading files in this workspace, writing memory, and writing `outbox/` drafts are the actions that may proceed under that profile when the tool risk allows it. Credentials are not stored in this directory.
