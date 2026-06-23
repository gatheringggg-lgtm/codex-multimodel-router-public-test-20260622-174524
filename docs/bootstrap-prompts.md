# Bootstrap prompts

Replace `gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524` with the actual GitHub repository.

## Install the skill

```text
请使用 skill-installer 从 GitHub 安装这个 skill：
https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/tree/main/skills/codex-multimodel-router-installer

安装完成后提醒我重启 Codex。
```

After installation, fully quit and reopen Codex Desktop.

## Start router installation

```text
请使用 codex-multimodel-router-installer skill，
优先从 Gitee 国内镜像 https://gitee.com/leo391913/codex-multimodel-router-public-test-20260622-174524/releases/tag/v0.1.3-rc1 下载适合本机平台的安装包；如果 Gitee 不可用，再 fallback 到 GitHub https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/tag/v0.1.3-rc1，
校验 SHA256，
然后按 interruption-safe 协议引导我安装 Codex Desktop 多模型 router。

要求：
- 不读取、不打印 API key；
- 不直接修改 Codex 配置；
- 使用包内 scripts；
- 先运行包内 `00-create-desktop-entry`，创建桌面 `配置多模型 Router`，然后让我通过这个本地中文页面配置 Base URL 和 API Key、生成安装方案、创建一键回滚、应用配置、安装和启动 Router；不要让我手工复制 `router.env.example`，除非本地页面打不开；
- final apply 前必须由 Codex/脚本自动确认 rollback 安全文件和桌面 `一键回滚` 已生成，不要让我自己找文件；
- final apply 前必须提醒我当前 Codex 对话可能中断；如果异常，我应双击桌面 `一键回滚`，完全退出并重新打开 Codex Desktop，再联系管理员/支持。
```
