# Sentriex integration skill

An integration workflow for Claude Code and OpenAI Codex, with the merchant-facing Public API contract, CLI reference, callback verification guide, and request examples.

[繁體中文](README.zh-TW.md)

## Install

Install the CLI after its first release is published. Go and Node.js are not required.

### macOS and Linux

Use the official Homebrew tap:

```sh
brew install heaven-online/tap/sentriex
sentriex --version
```

### Windows

Download `sentriex_v0.1.0_windows_amd64.zip` (x64) or `sentriex_v0.1.0_windows_arm64.zip` (ARM64) and `checksums.txt` from the [official Releases](https://github.com/heaven-online/sentriex-agent-skills/releases). Compare the ZIP's `Get-FileHash -Algorithm SHA256` result with its entry in `checksums.txt`.

For x64, extract and run in PowerShell:

```powershell
$sentriexZip = "$env:USERPROFILE\Downloads\sentriex_v0.1.0_windows_amd64.zip"
$sentriexInstallDir = "$env:USERPROFILE\Tools\sentriex"
Expand-Archive -Path $sentriexZip -DestinationPath $sentriexInstallDir -Force
& "$sentriexInstallDir\sentriex.exe" --version
$env:Path = "$sentriexInstallDir;$env:Path"
```

For ARM64, use the ARM64 ZIP name. The `Path` change applies to this terminal session; add the installation folder to your user `Path` in Windows Environment Variables for future terminals. Without a `Path` entry, call the executable by its full path.

### Install the skill

Inside your application repository, install the skill for your coding agent:

```sh
sentriex skills install --agent claude-code
# Or:
sentriex skills install --agent codex
```

Use `--global` for a user-wide installation, `--ref <tag-or-commit>` for a specific skill revision, and `--force` to replace an existing installation. The CLI installer requires neither Node.js nor Git.

The skill can also be installed independently through the skills ecosystem:

```sh
npx skills add heaven-online/sentriex-agent-skills --skill sentriex-integration
```

These commands target **Claude Code and Codex**. They do not install extensions into the ordinary Claude or ChatGPT chat interfaces.

## Start in sandbox

Set `SENTRIEX_BASE_URL` to the actual Public API origin and supply your platform's `sk_test_` API key through `SENTRIEX_API_KEY`. Obtain these from your Sentriex onboarding contact and merchant settings. Do not put keys in prompts or tracked files.

```sh
sentriex doctor --json
```

Ask your coding agent: "Use the Sentriex integration skill to implement sandbox deposits and signed callbacks in this application."

Supported contract: Public API **0.3.0**. CLI minimum version: **0.1.0**. The API key selects merchant, platform, and sandbox/production scope; the host name does not select that business scope.

The skill guides application code development. The CLI is a development and diagnostic tool, not a required subprocess for production application requests.
