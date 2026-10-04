# Sentriex CLI reference

Minimum CLI version: 0.1.0. Public API contract: 0.3.0.

## Install the CLI

After the first CLI release is published, macOS and Linux users can run `brew install heaven-online/tap/sentriex`.

On Windows, download the x64 `sentriex_v0.1.0_windows_amd64.zip` or ARM64 `sentriex_v0.1.0_windows_arm64.zip` from the [official Releases](https://github.com/heaven-online/sentriex-agent-skills/releases). Verify the ZIP's SHA256 against the release's `checksums.txt`, then extract it. The ZIP contains `sentriex.exe` and dependency license notices; Go, Node.js, and Git are not needed to run it.

Example for x64 in PowerShell:

```powershell
$sentriexZip = "$env:USERPROFILE\Downloads\sentriex_v0.1.0_windows_amd64.zip"
Get-FileHash -Path $sentriexZip -Algorithm SHA256
$sentriexInstallDir = "$env:USERPROFILE\Tools\sentriex"
Expand-Archive -Path $sentriexZip -DestinationPath $sentriexInstallDir -Force
& "$sentriexInstallDir\sentriex.exe" --version
$env:Path = "$sentriexInstallDir;$env:Path"
```

Use the ARM64 filename on ARM64. Adding the folder to `Path` above affects only the current terminal; add it to the Windows user `Path` for future terminals. You can also invoke the executable by its full path. Install project skills from the developer's application directory.

## Configuration

| Setting | Meaning |
|---|---|
| `SENTRIEX_BASE_URL` / `--base-url` | Public API origin without a path; HTTPS, or HTTP loopback for local testing |
| `SENTRIEX_API_KEY` | Platform-scoped bearer key; never pass as a command argument |
| `SENTRIEX_ENVIRONMENT` / `--environment` | `sandbox` (default) or `production`; must match the key prefix |
| `SENTRIEX_CALLBACK_SECRET` | Separate callback signing secret, used only for local callback verification |

The business environment is selected by the API key, independently of the deployment host. CLI environment configuration checks that selection; it is not a scope field sent to the API.

PowerShell uses `$env:SENTRIEX_BASE_URL` and `$env:SENTRIEX_ENVIRONMENT` to set these variables. Load `SENTRIEX_API_KEY` and callback secrets from the developer's local secret provider, without putting them in chat, scripts, or tracked files.

## Commands

```sh
sentriex --version
sentriex doctor --json

sentriex deposits create --input deposit.json --idempotency-key app-payment-123 --dry-run --json
sentriex deposits create --input deposit.json --idempotency-key app-payment-123 --json
sentriex deposits get <deposit-order-id> --json

sentriex withdrawals create --input withdrawal.json --idempotency-key app-withdrawal-123 --dry-run --json
sentriex withdrawals create --input withdrawal.json --idempotency-key app-withdrawal-123 --json
sentriex withdrawals get <withdrawal-order-id> --json

sentriex callbacks verify --body callback.json --headers headers.json --json
sentriex callbacks verify --body callback.json --headers headers.json --signature-only --json
```

Use `--input -` to read an unchanged JSON body from stdin. Order creates send the original file bytes, including whitespace, and sign the same bytes. Input must be a JSON object of at most 1 MiB. Creates are not automatically retried. Redirects are not followed. Preserve the input and idempotency key for retries.

On Windows, prefer `--input <file>` and UTF-8 JSON files without a BOM. Windows PowerShell 5.1's default redirection and text-writing commands can change encoding; do not pipe captured callback bodies through `Get-Content` or rewrite them before verification.

`doctor` verifies configuration and public health only. It explicitly reports `authentication_verified: false`. A sandbox GET for an existing order verifies authentication; it does not verify permission to create orders or fund availability.

`--dry-run` makes no request. It does not confirm that the server accepts the payload, asset, limits, balance, or destination.

Successful API calls return an envelope:

```json
{"status":201,"request_id":"req_example","idempotency_key":"app-payment-123","data":{"deposit_order_id":"dep_example","gross_amount":"1.25"}}
```

The `data` object follows the bundled OpenAPI schema. Errors are JSON on stderr with `code`, `message`, and optional `status`, `request_id`, and `retry_after`. Known secrets are redacted.

| Exit | Meaning |
|---|---|
| 0 | Command succeeded; a create success is not proof of financial completion |
| 1 | Output could not be written |
| 2 | Invalid arguments, local input, or configuration |
| 3 | API returned a non-2xx response |
| 4 | Network failure or unreadable/invalid successful response |
| 5 | Callback signature, timestamp, or event identity verification failed |
| 6 | Skill download or installation failed |

On 429, respect `retry_after`. On uncertain create outcomes, retain the same body/key. Handle `idempotency_key_reused`, `merchant_disabled`, `platform_disabled`, `invalid_api_signature`, `request_timestamp_out_of_window`, `insufficient_pool`, and other codes according to the contract. Do not infer safe retries merely from an HTTP status.

## Install/update the skill

```sh
sentriex skills install --agent claude-code
sentriex skills install --agent codex
sentriex skills install --agent codex --global
sentriex skills install --agent claude-code --ref <tag-or-commit> --force
```

| Agent | Project directory | User-wide directory |
|---|---|---|
| Claude Code | `.claude/skills/sentriex-integration` | `~/.claude/skills/sentriex-integration` |
| Codex | `.agents/skills/sentriex-integration` | `~/.codex/skills/sentriex-integration` |

On Windows, `~` is the user profile directory: user-wide skills go to `$env:USERPROFILE\.claude\skills\sentriex-integration` or `$env:USERPROFILE\.codex\skills\sentriex-integration`.

The installer downloads only the named skill from the official public repository. It checks archive bounds and paths, stages a complete installation, and requires `--force` before replacing an existing directory. It never replaces a target symlink or file. `--ref` defaults to `main`; use a tag or commit when you need a fixed revision. Standard agent loading rules still apply; reopen the agent session if needed.
