# Customer install flow

This is the expected customer-facing flow after the skill is installed.

## 1. Download and verify

The skill chooses the correct public release asset for the customer's platform, prefers the Gitee domestic mirror for large zips, falls back to GitHub if needed, and verifies SHA256 before extraction.

Current assets:

- Windows x64: `codex-multimodel-router-windows-x64-v24.17.0.zip`
- macOS Apple Silicon: `codex-multimodel-router-macos-arm64-v24.17.0.zip`

If SHA256 verification fails, stop.

## 2. Local env setup

The customer copies `config/router.env.example` to `config/router.env` and fills API settings locally.

The install guide must be beginner-friendly here: show the exact `router.env` path, show a placeholder-only template, explain Base URL vs provider key fields, and ask the customer to save locally and reply only `已填好`.

Secrets must stay local. The customer should not paste API keys into Codex chat.

## 3. Read-only checks

Run package scripts:

- `00-doctor`
- `01-detect`
- `02-dry-run`

The generated candidate config must not define `[model_providers.openai]`.

## 4. Rollback before apply

Run `03-emit-apply-rollback`.

Before final apply, the agent/script must verify these non-secret safety artifacts. Do not ask a beginner customer to manually confirm file existence:

- `outputs/apply.*`
- `outputs/rollback.*`
- Desktop `一键回滚` and backup `ROLLBACK-FIRST.*`
- `outputs/install-state.json`

## 5. Final apply warning

Before `04-apply-reviewed`, the skill must warn:

```text
Final apply may interrupt this Codex conversation. Keep the terminal open. I have checked the rollback safety files and the Desktop entry named `一键回滚`. If Codex looks abnormal, double-click `一键回滚` on the Desktop, then fully quit/reopen Codex Desktop and contact the administrator/support person who provided this package. Validate only after restart.
```

## 6. Restart and smoke test

After final apply and startup setup, fully quit and reopen Codex Desktop.

Smoke test:

- left history sidebar visible;
- picker shows GPT and selected third-party models;
- GPT route works;
- each third-party model completes a tiny no-file-change tool task;
- no raw `<function_calls>` or `<tool_call>` text appears.
