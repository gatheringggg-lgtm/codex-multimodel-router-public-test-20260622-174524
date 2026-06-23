# Customer install flow

This is the expected customer-facing flow after the skill is installed.

## 1. Download and verify

The skill chooses the correct public release asset for the customer's platform, prefers the Gitee domestic mirror for large zips, falls back to GitHub if needed, and verifies SHA256 before extraction.

Current assets:

- Windows x64: `codex-multimodel-router-windows-x64-v24.17.0.zip`
- macOS Apple Silicon: `codex-multimodel-router-macos-arm64-v24.17.0.zip`

Windows prototype path note: the Windows installer flow expects the extracted package to live at `D:\CodexMultiModelRouter`. This is intentional to avoid the system drive in the current prototype. If the customer has no usable `D:` drive, stop and use a dynamic-path build later; do not silently move the package and continue.

If SHA256 verification fails, stop.

## 2. Bootstrap local installer page

The package already includes or auto-creates `config/router.env`. The customer should not copy `config/router.env.example` by hand.

Default flow: create a Desktop entry first, then let the local page own the rest of the install.

- Windows PowerShell: `.\scripts\00-create-desktop-entry.ps1`
- macOS: `./scripts/00-create-desktop-entry.sh`

After this, the Desktop should have `配置多模型 Router`. The customer double-clicks it and uses the local Chinese page.

## 3. Configure, plan, rollback, apply, startup

Use the page buttons in order. Later steps are locked until earlier steps succeed:

1. `保存配置`
2. `生成安装方案`
3. `创建一键回滚`
4. `应用到 Codex`
5. `安装后台 Router`
6. `启动 Router`
7. `健康检查`
8. `完全重启 Codex`

The page lets the customer confirm Base URL fields and fill API Key fields locally. It must not print saved API Key values back to chat. The generated candidate config must not define `[model_providers.openai]`.

Manual `config/router.env` editing is fallback only if the browser page cannot open. In that fallback, show the exact `router.env` path, show a placeholder-only template, explain Base URL vs provider key fields, and ask the customer to save locally and reply only `已填好`.

Secrets must stay local. The customer should not paste API keys or full Base URLs into Codex chat.

## 4. Rollback before apply

Before final apply, the page/script must verify these non-secret safety artifacts. Do not ask a beginner customer to manually confirm file existence:

- `outputs/apply.*`
- `outputs/rollback.*`
- Desktop `一键回滚` and backup `ROLLBACK-FIRST.*`
- `outputs/install-state.json`

## 5. Final apply warning

Before local page/script final apply, the skill must warn:

```text
Final apply may interrupt this Codex conversation. Keep the terminal open. I have checked the rollback safety files and the Desktop entry named `一键回滚`. If Codex looks abnormal, double-click `一键回滚` on the Desktop, then fully quit/reopen Codex Desktop and contact the administrator/support person who provided this package. Validate only after restart.
```

## 6. Restart and smoke test

After final apply and startup setup, use the page button `完全重启 Codex` when available, or fully quit and reopen Codex Desktop manually.

Smoke test:

- left history sidebar visible;
- picker shows GPT and selected third-party models;
- GPT route works;
- each third-party model completes a tiny no-file-change tool task;
- no raw `<function_calls>` or `<tool_call>` text appears.
