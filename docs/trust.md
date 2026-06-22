# Trust notes

This helper is not a modified Codex Desktop distribution.

## What it does

- Installs a local loopback router.
- Generates a reviewable Codex config candidate.
- Creates a timestamped config backup before final apply.
- Creates `ROLLBACK-FIRST.*` and a Desktop emergency entry named `一键回滚` before final apply.
- Adds a user-level startup item only after explicit customer action.

## What it does not do

- Does not include Codex Desktop.
- Does not modify Codex Desktop app binaries.
- Does not edit conversation storage.
- Does not edit auth files, cookies, or session history.
- Does not require the customer to paste API keys into chat.

## Secret handling

Provider credentials are entered only into local `config/router.env` by the customer. The skill and docs should discuss only whether a route is configured, never the secret value.
