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

Download `sentriex_<version>_windows_amd64.zip` (x64) or `sentriex_<version>_windows_arm64.zip` (ARM64) and `checksums.txt` from the [official Releases](https://github.com/heaven-online/sentriex-agent-skills/releases), where `<version>` is the release tag shown on the Releases page, such as `v0.2.0`. Compare the ZIP's `Get-FileHash -Algorithm SHA256` result with its entry in `checksums.txt`.

For x64, extract and run in PowerShell:

```powershell
$sentriexVersion = "v0.2.0" # Replace with the version you downloaded
$sentriexZip = "$env:USERPROFILE\Downloads\sentriex_${sentriexVersion}_windows_amd64.zip"
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

Run this once in your own terminal (on Windows, PowerShell or Windows Terminal):

```sh
sentriex configure
```

Press Enter at the base URL prompt to accept the default, `https://api.sentriex.io`, then paste your platform's `sk_test_` API key from merchant settings; the key is not echoed. The callback secret is optional and is used only to verify captured callbacks locally. The settings are saved for your user account in `~/.config/sentriex/config.json`; on macOS and Linux, only you can read the file. A coding agent cannot run this command for you; do not put keys in prompts or tracked files.

Then check the configuration:

```sh
sentriex doctor --json
```

For CI or a temporary override, set `SENTRIEX_API_KEY`, `SENTRIEX_BASE_URL`, `SENTRIEX_ENVIRONMENT`, or `SENTRIEX_CALLBACK_SECRET` instead. Command flags take precedence over environment variables, which take precedence over the configuration file and the built-in default; `doctor` reports where each value came from.

Ask your coding agent: "Use the Sentriex integration skill to implement sandbox deposits and signed callbacks in this application."

Supported contract: Public API **0.3.0**. CLI minimum version: **0.2.0**. The API key selects merchant, platform, and sandbox/production scope; the host name does not select that business scope.

The skill guides application code development. The CLI is a development and diagnostic tool, not a required subprocess for production application requests.
