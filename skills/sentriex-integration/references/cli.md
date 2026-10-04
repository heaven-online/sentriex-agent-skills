# Sentriex CLI reference

Minimum CLI version: 0.1.0. Public API contract: 0.3.0.

## Configuration

| Setting | Meaning |
|---|---|
| `SENTRIEX_BASE_URL` / `--base-url` | Public API origin without a path; HTTPS, or HTTP loopback for local testing |
| `SENTRIEX_API_KEY` | Platform-scoped bearer key; never pass as a command argument |
| `SENTRIEX_ENVIRONMENT` / `--environment` | `sandbox` (default) or `production`; must match the key prefix |
| `SENTRIEX_CALLBACK_SECRET` | Separate callback signing secret, used only for local callback verification |

The business environment is selected by the API key, independently of the deployment host. CLI environment configuration checks that selection; it is not a scope field sent to the API.

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

The installer downloads only the named skill from the official public repository. It checks archive bounds and paths, stages a complete installation, and requires `--force` before replacing an existing directory. It never replaces a target symlink or file. `--ref` defaults to `main`; use a tag or commit when you need a fixed revision. Standard agent loading rules still apply; reopen the agent session if needed.
