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

Confirm the package is for the customer's own Codex Desktop on macOS or Windows. Explain briefly:

- this is not a modified Codex distribution;
- the skill/Agent only bootstraps the installer;
- the local installer page will own provider setup, rollback creation, final apply, startup install, health check, and logs;
- final apply may interrupt the current Codex conversation, so rollback must exist on the Desktop before apply.

### 2. Bootstrap the local installer page

Default route: create a Desktop entry named `配置多模型 Router`, then stop and tell the customer to open it.

Run from the extracted package root:

- macOS: `./scripts/00-create-desktop-entry.sh`
- Windows PowerShell: `.\scripts\00-create-desktop-entry.ps1`

After it succeeds, tell the customer in beginner-friendly Chinese:

```text
桌面上已经创建了“配置多模型 Router”。
请双击它打开本机配置页面。
在页面里按顺序点击：保存配置 → 生成安装方案 → 创建一键回滚 → 应用到 Codex → 安装后台 Router → 启动 Router → 健康检查。
API Key 只在本机页面填写，不要发到聊天里。
如果应用后 Codex 异常，先双击桌面“一键回滚”，然后完全退出并重新打开 Codex Desktop，再联系管理员/支持。
```

Do not ask a beginner customer to manually copy `router.env.example`, inspect `install-state.json`, or confirm rollback files. The local page/scripts should perform those checks.

### 3. Provider setup inside the local page

The local page must be preferred over manual file editing. It lets the customer:

1. Fill or confirm `中转站 Base URL`.
2. Keep `GPT / Codex 透传地址与中转站地址相同` checked for common CC Switch / Packy / relay-login scenarios.
3. Fill `DeepSeek 分组 Key` and `MiMO 分组 Key`.
4. Click `保存配置`.

The page must not display already-saved API key values. Blank key inputs preserve existing keys.

### 4. Local page install sequence

The customer should use the page buttons, not chat-driven final scripts, unless troubleshooting requires fallback:

1. `保存配置`
2. `生成安装方案`
3. `创建一键回滚`
4. `应用到 Codex`
5. `安装后台 Router`
6. `启动 Router`
7. `健康检查`
8. `查看日志` only if health fails

Before `应用到 Codex`, the page/package must have already created Desktop `一键回滚`. State and logs must not contain secrets.

### 5. CLI fallback only

Use CLI fallback only if the Desktop entry or browser page cannot open.

Fallback local page command:

- macOS: `./scripts/00-configure.sh`
- Windows PowerShell: `.\scripts\00-configure.ps1`

Manual `config/router.env` editing is last resort. If required, give a short Chinese action block:

1. Open the exact local file path:
   - Windows: `D:\CodexMultiModelRouter\config\router.env`
   - macOS: `<package-root>/config/router.env`
2. Use a local editor such as Notepad, VS Code, or TextEdit plain text mode.
3. Replace only placeholder text after `=`. Do not paste values into chat.
4. Placeholder-only template:

```text
PACKY_BASE_URL=https://YOUR_PROVIDER_BASE_URL/v1
CODEX_OFFICIAL_BASE_URL=https://YOUR_PROVIDER_BASE_URL/v1
PACKY_API_KEY_DEEPSEEK=PASTE_DEEPSEEK_GROUP_KEY_HERE
PACKY_API_KEY_MIMO=PASTE_MIMO_GROUP_KEY_HERE
```

Plain-language field meanings:

- `PACKY_BASE_URL`: 中转站/供应商 Base URL，通常以 `/v1` 结尾。安装器会尽量从当前 Codex 配置自动抓取。
- `CODEX_OFFICIAL_BASE_URL`: GPT/Codex 官方模型透传 Base URL。国内 CC Switch / Packy / custom provider 场景通常和 `PACKY_BASE_URL` 一样。
- `PACKY_API_KEY_DEEPSEEK`: 可调用 DeepSeek 的分组 Key。
- `PACKY_API_KEY_MIMO`: 可调用 MiMO 的分组 Key。

### 6. Final apply wording if using CLI fallback

If fallback scripts force a chat-guided final apply, say plainly:

```text
Final apply may interrupt this Codex conversation. Keep the terminal open. I have checked the rollback safety files and the Desktop entry named 一键回滚. If Codex looks abnormal, double-click 一键回滚 on the Desktop, then fully quit/reopen Codex Desktop and contact the administrator/support person who provided this package. Validate only after restart.
```

### 7. Smoke test

Use `docs/post-install-smoke-test.md`. Required checks:

- left history sidebar visible;
- picker shows GPT and selected third-party models;
- GPT route keeps expected reasoning options;
- each third-party model completes a tiny no-file-change tool task;
- transcript does not show raw `<function_calls>`, `<tool_call>`, or XML-like tool markup.

### 8. Troubleshooting and rollback

If history disappears, picker is wrong, error banners appear, GPT route returns Bad Gateway, or raw tool-call text appears, stop expansion.

Rollback options:

- beginner emergency rollback after final apply: double-click `一键回滚` on the Desktop;
- backup fast config rollback: `ROLLBACK-FIRST.*` in the package root;
- full rollback/uninstall: `scripts/99-rollback-all.*`.

Never advise deleting Codex history or session files.
