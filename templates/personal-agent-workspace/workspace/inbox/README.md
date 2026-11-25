Drop one JSON file per event when you want the filesystem channel to wake the agent.

The sample config watches `*.json` in this directory. It is off until you enable `[channels.filesystem.inbox]` and point `paths` here. Email and the webhook channel are the other inboxes; they do not use this folder.
