# Codex Multi-Model Router Installer

This repository distributes a skill-guided helper for a user's own Codex Desktop installation.

It does **not** include Codex Desktop. It does **not** modify Codex Desktop app binaries, auth files, cookies, or conversation history. It installs a local loopback router and a review-first config helper so GPT routes and selected third-party API routes can appear in the same Codex Desktop model picker.

## Customer quick start

### Step 1: install the skill

Ask Codex:

```text
请使用 skill-installer 从 GitHub 安装这个 skill：
https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/tree/main/skills/codex-multimodel-router-installer

安装完成后提醒我重启 Codex。
```

Then fully quit and reopen Codex Desktop so the new skill is loaded.

### Step 2: let the skill guide the router install

Ask Codex:

```text
请使用 codex-multimodel-router-installer skill，
优先从 Gitee 国内镜像 https://gitee.com/leo391913/codex-multimodel-router-public-test-20260622-174524/releases/tag/v0.1.3-rc1 下载适合本机平台的安装包；如果 Gitee 不可用，再 fallback 到 GitHub https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/tag/v0.1.3-rc1，
校验 SHA256，
然后按 interruption-safe 协议引导我安装 Codex Desktop 多模型 router。

要求：
- 不读取、不打印 API key；
- 不直接修改 Codex 配置；
- 先运行包内 `00-create-desktop-entry`，在桌面创建 `配置多模型 Router`；
- 让我通过桌面 `配置多模型 Router` 本地中文页面填写 Base URL 和 API Key、生成方案、创建一键回滚、应用配置、安装和启动 Router；
- 不要让我手工复制 `router.env.example`，除非本地页面打不开；
- final apply 前必须由本地页面/脚本自动确认 rollback 安全文件和桌面 `一键回滚` 已生成，不要让我自己找文件；
- final apply 前必须提醒我当前 Codex 对话可能中断；如果异常，我应双击桌面 `一键回滚`，完全退出并重新打开 Codex Desktop，再联系管理员/支持。
```

## Why two steps?

Codex normally needs a restart after installing a new skill. The router's final apply step can also require Codex Desktop to be fully quit and reopened. The installer is designed around this reality: all critical recovery state is written to local files before final apply.

## Release assets

Release `v0.1.3-rc1` should include:

```text
codex-multimodel-router-windows-x64-v24.17.0.zip
codex-multimodel-router-macos-arm64-v24.17.0.zip
SHA256SUMS.txt
release-manifest.json
```

See `release-manifest.json` and `docs/release-publishing.md`.

## Supported platforms in current prototype

- Windows x64
- macOS Apple Silicon

Windows prototype install path: the current Windows package is designed to be placed at `D:\CodexMultiModelRouter` so it avoids the system drive by default. Make sure the Windows machine has a usable `D:` drive before asking the skill to proceed. If the customer does not have a `D:` drive, stop and use a later dynamic-path build instead of improvising paths manually.

## Current RC notes

- The local configuration page uses a locked step-by-step flow and a final `完全重启 Codex` button.
- Windows startup uses user-level S4U background task mode so the Router keeps running after Codex/Agent/terminal sessions end.

## Safety model

- Customer provider credentials stay in local `config/router.env`.
- The skill should never ask the customer to paste secrets into chat.
- The package writes `ROLLBACK-FIRST.*`, `outputs/install-state.json`, and a Desktop emergency entry named `一键回滚` before final apply.
- If Codex looks abnormal after final apply, double-click `一键回滚` on the Desktop, then fully quit and reopen Codex Desktop, and contact the administrator/support person who provided this package.

More details: `docs/trust.md` and `docs/customer-install-flow.md`.
