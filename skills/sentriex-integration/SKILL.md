---
name: sentriex-integration
description: Integrate an application with Sentriex Public API deposits, withdrawals, signed merchant callbacks, and sandbox testing using the official Sentriex CLI. Use when a developer asks to connect their application to Sentriex or troubleshoot its public integration.
metadata:
  api-version: "0.3.0"
  cli-min-version: "0.1.0"
---

# Sentriex integration

Help the developer implement Sentriex in their own server-side application. Respond in the developer's language. Use the bundled public contract and references; never assume access to Sentriex's private repository, internal API, databases, or ops CLI.

## References

- [Public API contract](references/public-api.yaml): endpoint and schema authority.
- [CLI operations](references/cli.md): setup, commands, output, and exit codes.
- [Merchant callbacks](references/callbacks.md): event envelope, authentication, deduplication, and retries.
- [Deposit request template](examples/deposit.json).
- [Withdrawal request template](examples/withdrawal.json).

## Workflow

1. Inspect the developer's application and identify its server language, framework, persistence, and existing payment flows. Confirm whether they want deposits, withdrawals, or both; use the platform's enabled chain/token identifiers. The Public API does not offer an asset-listing or platform-configuration endpoint.
2. Check `sentriex --version` (minimum 0.1.0). If unavailable, use `brew install heaven-online/tap/sentriex` when Homebrew is available; otherwise use an official binary for the host platform. A developer can supply a binary already obtained through their organization. Skill installation alone does not install the CLI. The CLI skill installer needs no Node.js.
3. Start with sandbox. Configure the actual Public API origin through `SENTRIEX_BASE_URL` or `--base-url`, and a platform-scoped `sk_test_` key through `SENTRIEX_API_KEY`. Keys should be loaded by the local environment or secret manager; do not ask the developer to paste a key into chat or print it. `sk_live_` selects production and requires an explicit production configuration. Merchant, platform, and environment scope derive from the key; never add scope fields to request bodies.
4. Run `sentriex doctor --json`. This checks key format, environment consistency, and unauthenticated health; it does not establish API key validity, permissions, IP admission, or funded balances. Retrieve a known order to verify authentication, or create a sandbox order when the developer has authorized that test. Record the machine error code and request ID when a call fails.
5. Implement the application client. Retain decimal amount strings, serialize each request body once, then sign and transmit those exact bytes. Reads require bearer authentication. Creates also require a Unix timestamp, HMAC signature, and a stable idempotency key. Use the key's secret component for request HMAC, not the full bearer token. See the contract for the canonical string.
6. For deposits, create a fixed or open order; fixed requires `gross_amount`, open omits it. Correlate `merchant_reference` with the application's payment record. Return the provided cashier URL to the user. A created or detected order is not credited funds. Apply credit only after an authenticated credited event or confirmed credited order, using the application's own durable idempotent transaction.
7. For withdrawals, enforce the application's end-user authorization and limits before creating an order. Replace the request template's destination with the developer's intended sandbox address. `processing` is not completion. Handle completed, failed, and reversal-pending outcomes according to the contract and callback guide. Do not initiate a production payment unless the developer has explicitly authorized that operation.
8. Implement the callback route using the original raw body bytes. Use the separate environment-specific callback secret, check the timestamp and signature in constant time, match header/body event IDs, and deduplicate by `event_id` in durable storage. Only acknowledge with 2xx after the event is durably accepted. Authenticate duplicate deliveries before acknowledging them; do not apply their financial effects again. Production/sandbox secrets are independent.
9. Exercise the implemented application and compare with official CLI sandbox requests. The CLI can create/retrieve orders and verify captured callbacks; it cannot fund a wallet, confirm a chain transaction, configure callbacks, or replay server deliveries. Actual callback delivery requires a funded sandbox transaction and a public HTTPS callback endpoint configured in merchant settings. Localhost and private-IP callback endpoints are rejected by the platform.
10. Test failed signatures, stale timestamps, duplicate events, same-key request retries, mismatched payload reuse, and order state transitions relevant to the application. If a real sandbox endpoint, funds, or credentials are unavailable, use fixtures and state precisely which live checks remain unverified.

## Operation rules

- A timeout does not prove an order was rejected. Retain the same input bytes and idempotency key when retrying the same operation. Do not generate a new key for an uncertain result.
- Use a new key only for a deliberately new operation. The same key with a different payload is rejected.
- Use `--dry-run` to preview a create operation; it validates local configuration and does not validate the payload against server-side asset settings.
- For automated CLI calls, use `--json`; success goes to stdout and structured failures to stderr. Branch on exit code and the stable `code` field, not translated message text.
- Never put API keys in browser code, URLs, command arguments, source control, logs, or prompts. CLI authentication comes from environment variables.
- Never use `callbacks verify --signature-only` to accept live callbacks. It is for diagnosing a captured historical delivery.
- Produce application code, relevant tests, and a clear description of verified behavior and outstanding live checks. The production application calls the API directly; it need not execute the CLI.
