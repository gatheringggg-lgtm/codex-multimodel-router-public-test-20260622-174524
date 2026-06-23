---
name: codex-multimodel-router-installer
description: Guide interruption-safe download, installation, validation, and rollback of the local multi-model router for a user's own Codex Desktop. Use when the user wants a skill-guided Release download, local package install, dry run, final apply, startup setup, smoke test, troubleshooting, or rollback on macOS or Windows.
---

# Codex Multi-Model Router Installer

## Overview

Use this skill to guide a customer through installing the local multi-model router for their own Codex Desktop without distributing or modifying Codex Desktop itself.

The skill is a coach, not the executor. It should instruct the customer to run package scripts, inspect outputs, and approve gates. The scripts perform stateful local changes and write rollback files.

## Hard safety rules

- Never ask the customer to paste API keys, bearer tokens, cookies, auth files, or session contents into chat.
- Never print secret values from `router.env`; only discuss whether a required setting is configured.
- Do not edit Codex config directly from the skill. Use package scripts.
- Do not modify Codex app binaries, conversation storage, auth files, cookies, or session/history files.
- Treat `scripts/04-apply-reviewed.*` as the final apply gate because it may interrupt the current Codex conversation.
- If the customer is installing the skill from GitHub, tell them to restart Codex after skill installation before continuing router installation.

## Documents to use

From the package root, read only the docs needed for the current phase:

- `README-FIRST.md` for the customer install flow.
- `docs/interruption-safe-install.md` before final apply.
- `docs/post-install-smoke-test.md` after restart.
- `docs/troubleshooting.md` when UI, picker, history, routing, or tool-call behavior is abnormal.
- `manifest.json` to confirm platform, service name, port, and package contents.

From this skill, use:

- `references/download-release.md` when the customer wants Codex to fetch the package from public release assets.
- `references/install-protocol.md` for the final apply and rollback phase contract.

## Workflow decision

1. If the customer already has an extracted package path, start at **Local package orientation**.
2. If the customer gives a Gitee/GitHub repo or release URL, start at **Release download**.
3. If the customer only installed this skill and does not know what to do next, ask for the Gitee/GitHub repo URL or the local package path.

## Release download

Use `references/download-release.md`.

High-level rule:

- determine platform;
- download only the matching release asset and checksum/manifest;
- verify SHA256 before extraction;
- extract to a normal user-writable folder;
- then continue with the local package workflow.

Do not download or run anything from an unverified archive. Do not ask the customer to paste secrets.

## Local package workflow

### 1. Orient

Confirm the package is for the customer's own Codex Desktop on macOS or Windows. Explain that this helper is not a modified Codex distribution and that final apply may require fully quitting and reopening Codex Desktop.

### 2. Prepare local provider settings

Do not tell the customer to copy `config/router.env.example` by hand. New packages already include `config/router.env`, and package scripts create it automatically if it is missing. First run detect/dry-run so the installer can auto-fill non-secret Base URL fields from the current Codex config. Then prefer the local configuration page instead of manual text editing:

- macOS: `./scripts/00-configure.sh`
- Windows PowerShell: `.\scripts\00-configure.ps1`

This opens a local browser page in Chinese for Base URL and API Key entry. Tell the customer:

1. Fill or confirm `中转站 Base URL`.
2. Keep `GPT / Codex 透传地址与中转站地址相同` checked for common CC Switch / Packy / relay-login scenarios.
3. Fill `DeepSeek 分组 Key` and `MiMO 分组 Key`.
4. Click `保存到本机`.
5. Return to the terminal and press `Ctrl+C` to close the local configuration page server.
6. Reply only `已保存` or `已填好`.

The page must not display already-saved API key values. Blank key inputs preserve existing keys.

Only use manual `config/router.env` editing as a fallback if the browser page cannot open. Do not ask customers to paste values into chat.

If `config/router.env` is missing, run `scripts/00-doctor.*` or `scripts/02-dry-run.*`; the package should create it automatically from `config/router.env.example`. If doctor says third-party keys or provider settings are missing, do not stop with a vague technical message. Give a Chinese beginner-friendly action block:

1. Show the exact local file path to open:
   - Windows: `D:\CodexMultiModelRouter\config\router.env`
   - macOS: `<package-root>/config/router.env`
2. Tell the customer to click/open that file in a text editor such as Notepad, VS Code, TextEdit plain text mode, or another local editor.
3. Show this safe template with placeholders only; never include real secrets:

```text
# 中转站 Base URL，通常以 /v1 结尾。安装脚本能识别时会自动填好。
PACKY_BASE_URL=https://YOUR_PROVIDER_BASE_URL/v1

# GPT / Codex 官方模型透传 Base URL。CC Switch / Packy 场景通常和 PACKY_BASE_URL 一样。
# 安装脚本能识别时会自动填好；没有自动填时才需要人工检查。
CODEX_OFFICIAL_BASE_URL=https://YOUR_PROVIDER_BASE_URL/v1

# 下面两个才是普通用户最常需要填写的 Key。
PACKY_API_KEY_DEEPSEEK=PASTE_DEEPSEEK_GROUP_KEY_HERE
PACKY_API_KEY_MIMO=PASTE_MIMO_GROUP_KEY_HERE
```

4. Explain the fields in plain language:
   - `PACKY_BASE_URL`: 中转站/供应商 Base URL，通常以 `/v1` 结尾。安装脚本会先尝试从当前 Codex 配置自动抓取。
   - `CODEX_OFFICIAL_BASE_URL`: GPT/Codex 官方模型透传 Base URL。国内 CC Switch / Packy / 其他 custom provider 场景下通常应和 `PACKY_BASE_URL` 完全一样。安装脚本会优先自动抓取；如果没有自动填，才让用户人工检查。官方 OpenAI 登录/provider 直连场景不要让小白自行乱填，转管理员/支持处理。
   - `PACKY_API_KEY_DEEPSEEK`: the API key or group key that can call DeepSeek.
   - `PACKY_API_KEY_MIMO`: the API key or group key that can call MiMO.
5. Tell the customer to replace only the placeholder text after `=` in their local file, save the file, and then reply only `已填好`. If the two Base URL lines are already real URLs, tell them not to touch those lines and only fill the API key placeholders.
6. Explicitly say: do not paste the Base URL or API keys into Codex chat. If they are unsure, they can paste a redacted shape such as `https://.../v1` or `sk-***last4`, never the full value.

### 3. Read-only checks

Ask the customer to run:

- macOS: `./scripts/00-doctor.sh`, then `./scripts/01-detect.sh`
- Windows PowerShell: `.\scripts\00-doctor.ps1`, then `.\scripts\01-detect.ps1`

Review only non-secret output. If doctor says a third-party route is not configured, route back to **Prepare local provider settings** and give the beginner-friendly action block. Do not ask the customer to paste secrets into chat.

### 4. Dry run

Ask the customer to run:

- macOS: `./scripts/02-dry-run.sh`
- Windows PowerShell: `.\scripts\02-dry-run.ps1`

Review `outputs/dry-run.json` and `outputs/candidate.config.toml` for expected provider mode. The candidate must not define `[model_providers.openai]`.

### 5. Generate rollback before apply

Ask the customer to run:

- macOS: `./scripts/03-emit-apply-rollback.sh`
- Windows PowerShell: `.\scripts\03-emit-apply-rollback.ps1`

The agent must verify these non-secret safety artifacts itself before final apply. Do not ask the customer to manually confirm file existence:

- `outputs/apply.*`
- `outputs/rollback.*`
- `ROLLBACK-FIRST.*`
- Desktop emergency entry named `一键回滚`
- `outputs/install-state.json`

`outputs/install-state.json` must show `phase: ready_for_final_apply`, `secretsStored: false`, and a `desktopRollbackPath` or equivalent desktop rollback entry. It must not contain secrets.

### 6. Final apply gate

Before instructing final apply, state plainly in beginner-friendly language:

- final apply may interrupt this Codex conversation;
- the terminal must stay open;
- the package/agent has already checked the rollback safety files and the Desktop entry named `一键回滚`;
- if Codex looks abnormal after the change, double-click `一键回滚` on the Desktop, then fully quit and reopen Codex Desktop;
- if rollback is needed, contact the administrator/support person who provided this package after using `一键回滚`;
- validation should happen after a full quit/reopen in a fresh or reloaded chat.

Then ask the customer to run:

- macOS: `./scripts/04-apply-reviewed.sh`
- Windows PowerShell: `.\scripts\04-apply-reviewed.ps1`

### 7. Startup setup

After final apply succeeds, ask the customer to run:

- macOS: `./scripts/05-install-startup.sh`
- Windows PowerShell: `.\scripts\05-install-startup.ps1`

Then fully quit and reopen Codex Desktop.

### 8. Smoke test

Use `docs/post-install-smoke-test.md`. Required checks:

- left history sidebar visible;
- picker shows GPT and selected third-party models;
- GPT route keeps expected reasoning options;
- each third-party model completes a tiny no-file-change tool task;
- transcript does not show raw `<function_calls>`, `<tool_call>`, or XML-like tool markup.

### 9. Troubleshooting and rollback

If history disappears, picker is wrong, error banners appear, or raw tool-call text appears, stop expansion. Use `docs/troubleshooting.md` and prefer rollback before more experiments.

Rollback options:

- beginner emergency rollback after final apply: double-click `一键回滚` on the Desktop;
- backup fast config rollback: `ROLLBACK-FIRST.*` in the package root;
- full rollback/uninstall: `scripts/99-rollback-all.*`.

Never advise deleting Codex history or session files.
