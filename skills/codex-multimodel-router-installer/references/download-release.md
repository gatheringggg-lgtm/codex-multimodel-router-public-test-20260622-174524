# GitHub Release download reference

Use this when the customer wants Codex to download the router package from GitHub Releases.

## Required inputs

Ask for one of these if not already provided:

- GitHub repo URL, such as `https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524`;
- GitHub release URL, such as `https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/tag/v0.1.0-prototype`;
- or an already downloaded/extracted package path.

Do not ask for API keys or provider credentials during download.

## Platform selection

Supported release assets in the current prototype:

| Customer platform | Asset name |
| --- | --- |
| Windows x64 | `codex-multimodel-router-windows-x64-v24.17.0.zip` |
| macOS Apple Silicon | `codex-multimodel-router-macos-arm64-v24.17.0.zip` |

If the platform is not Windows x64 or macOS arm64, stop and explain that the current package does not yet support that platform.

## Preferred download methods

Use one of these, depending on what is available.

### GitHub CLI

```bash
gh release download v0.1.0-prototype --repo gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524 --pattern 'codex-multimodel-router-macos-arm64-v24.17.0.zip' --dir ./codex-multimodel-router-download
gh release download v0.1.0-prototype --repo gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524 --pattern 'SHA256SUMS.txt' --dir ./codex-multimodel-router-download
```

Windows PowerShell example:

```powershell
gh release download v0.1.0-prototype --repo gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524 --pattern 'codex-multimodel-router-windows-x64-v24.17.0.zip' --dir .\codex-multimodel-router-download
gh release download v0.1.0-prototype --repo gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524 --pattern 'SHA256SUMS.txt' --dir .\codex-multimodel-router-download
```

### Direct public GitHub URL

macOS example:

```bash
mkdir -p ./codex-multimodel-router-download
curl -L -o ./codex-multimodel-router-download/codex-multimodel-router-macos-arm64-v24.17.0.zip \
  https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/download/v0.1.0-prototype/codex-multimodel-router-macos-arm64-v24.17.0.zip
curl -L -o ./codex-multimodel-router-download/SHA256SUMS.txt \
  https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/download/v0.1.0-prototype/SHA256SUMS.txt
```

Windows PowerShell example:

```powershell
New-Item -ItemType Directory -Force -Path .\codex-multimodel-router-download | Out-Null
Invoke-WebRequest -Uri 'https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/download/v0.1.0-prototype/codex-multimodel-router-windows-x64-v24.17.0.zip' -OutFile '.\codex-multimodel-router-download\codex-multimodel-router-windows-x64-v24.17.0.zip'
Invoke-WebRequest -Uri 'https://github.com/gatheringggg-lgtm/codex-multimodel-router-public-test-20260622-174524/releases/download/v0.1.0-prototype/SHA256SUMS.txt' -OutFile '.\codex-multimodel-router-download\SHA256SUMS.txt'
```

## SHA256 verification

macOS:

```bash
cd ./codex-multimodel-router-download
shasum -a 256 -c SHA256SUMS.txt --ignore-missing
```

Windows PowerShell:

```powershell
cd .\codex-multimodel-router-download
$Expected = (Select-String -Path .\SHA256SUMS.txt -Pattern 'codex-multimodel-router-windows-x64-v24.17.0.zip').Line.Split(' ')[0]
$Actual = (Get-FileHash -Algorithm SHA256 .\codex-multimodel-router-windows-x64-v24.17.0.zip).Hash.ToLowerInvariant()
if ($Actual -ne $Expected.ToLowerInvariant()) { throw "SHA256 mismatch" }
```

Do not continue if hash verification fails.

## Extraction

macOS:

```bash
mkdir -p ./codex-multimodel-router-install
unzip -q ./codex-multimodel-router-macos-arm64-v24.17.0.zip -d ./codex-multimodel-router-install
cd ./codex-multimodel-router-install/codex-multimodel-router-macos-arm64
```

Windows PowerShell:

The Windows package scripts are generated for this install root:

```text
D:\CodexMultiModelRouter
```

Use a staging folder, then move the extracted package root to that exact path:

```powershell
$InstallRoot = 'D:\CodexMultiModelRouter'
$StageRoot = 'D:\codex-multimodel-router-install-stage'
$BackupRoot = 'D:\CodexMultiModelRouterBackups'
$Stamp = Get-Date -Format yyyyMMdd-HHmmss
New-Item -ItemType Directory -Force -Path $BackupRoot | Out-Null
if (Test-Path -LiteralPath $InstallRoot) {
  Move-Item -LiteralPath $InstallRoot -Destination (Join-Path $BackupRoot "D-CodexMultiModelRouter-before-manual-install-$Stamp") -Force
}
Remove-Item -LiteralPath $StageRoot -Recurse -Force -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Force -Path $StageRoot | Out-Null
Expand-Archive -LiteralPath .\codex-multimodel-router-windows-x64-v24.17.0.zip -DestinationPath $StageRoot -Force
Move-Item -LiteralPath (Join-Path $StageRoot 'codex-multimodel-router-windows-x64') -Destination $InstallRoot -Force
cd $InstallRoot
```

After extraction, continue with the local package workflow in `SKILL.md`.
