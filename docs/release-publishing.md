# Release publishing checklist

Use this when publishing a new GitHub Release.

## Repository content

Commit these files to the repository:

```text
README.md
release-manifest.json
docs/
skills/codex-multimodel-router-installer/
```

Do not commit the platform zip packages to the main source tree. Upload them as GitHub Release assets.

## Release assets

Release tag:

```text
v0.1.1-rc1
```

Upload these assets:

```text
codex-multimodel-router-windows-x64-v24.17.0.zip
codex-multimodel-router-macos-arm64-v24.17.0.zip
SHA256SUMS.txt
release-manifest.json
```

## Pre-publish checks

- `release-manifest.json` matches the asset names and SHA256 values.
- `SHA256SUMS.txt` matches the uploaded zip files.
- Skill frontmatter validates.
- Customer docs do not contain secrets, local machine names, LAN IPs, internal paths, or development gate labels.
- Release notes state that final apply may interrupt the current Codex conversation, that rollback safety files and Desktop `一键回滚` are generated before apply, and that customers should use Desktop `一键回滚` first if Codex looks abnormal.

## Customer install command

After publishing, customers can install the skill by asking Codex:

```text
请使用 skill-installer 从 GitHub 安装这个 skill：
https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/tree/main/skills/codex-multimodel-router-installer

安装完成后提醒我重启 Codex。
```
