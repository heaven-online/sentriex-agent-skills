# Merchant callbacks

Merchant callbacks are separate from the platform's provider webhooks. Configure your application's **public HTTPS** callback URL and an environment-specific callback secret in merchant settings. The URL must resolve to public Internet addresses; localhost and private addresses are not accepted. To test a local application, provide an externally reachable HTTPS endpoint using your usual development infrastructure.

## Envelope and events

Every transaction event carries `event_id`, `event_type`, `api_version` (`v1`), `created_at` (RFC 3339), and `data`.

| Event | Data |
|---|---|
| `deposit_order.credited` | `deposit_order_id`, optional `merchant_reference`, `user_ref`, `asset`, `amount_mode`, `gross_amount`, `fee_amount`, `net_amount`, `ledger_journal_id`, and nullable `chain_transaction_hash`, `chain_transaction_id`, `provider_transaction_id` |
| `withdrawal_order.completed` | `withdrawal_order_id`, optional `merchant_reference`, `user_ref`, `asset`, `destination` (`address`, nullable `tag_or_memo`), `gross_amount`, `fee_amount`, `net_amount`, `ledger_journal_id`, `provider_transaction_id`, nullable `chain_transaction_hash`, and `completed_at` |
| `withdrawal_order.failed` | `withdrawal_order_id`, optional `merchant_reference`, `user_ref`, `asset`, `gross_amount`, `failure_code`, optional `provider_transaction_id`, and `reversal_status` (`reversed` or `reversal_pending`) |

Amounts are decimal strings. A missing chain hash is `null`; it is not an internal database identifier. In a completed withdrawal, `net_amount` is the on-chain transfer amount and `gross_amount = fee_amount + net_amount`.

An example credited event:

```json
{
  "event_id": "evt_example",
  "event_type": "deposit_order.credited",
  "api_version": "v1",
  "created_at": "2026-10-04T00:00:00Z",
  "data": {
    "deposit_order_id": "dep_example",
    "merchant_reference": "app-payment-123",
    "user_ref": "user-example",
    "asset": {"chain": "TRON", "token": "USDT"},
    "amount_mode": "fixed",
    "gross_amount": "10",
    "fee_amount": "1",
    "net_amount": "9",
    "ledger_journal_id": "journal_example",
    "chain_transaction_hash": null,
    "chain_transaction_id": null,
    "provider_transaction_id": null
  }
}
```

This illustration is not a signed test fixture; real signatures depend on the exact original bytes.

## Test callbacks

The merchant settings test sends `{"api_version":"v1","event_type":"callback.test","test":true}` without a body `event_id`. Only for this format with an absent `event_id`, use the nonempty `X-Sentriex-Event-Id` header as the event identity. Still verify the signature over the original body bytes and check timestamp freshness. If a body `event_id` is present, it must match the header; empty, null, and mismatched IDs are rejected. Transaction events must always include a matching body ID.

After successful authentication, acknowledge the test with 2xx without applying payment or ledger changes. It checks callback connectivity and authentication, not a real transaction.

## Authentication

Callback headers:

- `X-Sentriex-Event-Id`
- `X-Sentriex-Timestamp` (Unix seconds)
- `X-Sentriex-Signature` (`v1=` plus lowercase hex HMAC)

Calculate HMAC-SHA256 using the **callback secret**, with this canonical string:

```text
timestamp + "\n" + event_id + "\n" + sha256_hex(raw_body)
```

Compare signatures in constant time, match the header event ID to the body's `event_id` (except the test format described above), and validate freshness using the delivery timestamp header. The event's `created_at` is not the delivery timestamp: retries can deliver an older event with a fresh timestamp/signature.

The CLI uses a five-minute window around the local clock. Synchronize the receiver's clock and use the same window in integration examples. Keep the original body bytes before your framework parses JSON; serializing the parsed object again may produce a different signature.

For a captured delivery, save the raw body as `callback.json` and headers as a JSON object with string values:

```json
{
  "X-Sentriex-Event-Id": "evt_example",
  "X-Sentriex-Timestamp": "<delivery-unix-seconds>",
  "X-Sentriex-Signature": "v1=<64-hex-characters>"
}
```

Set `SENTRIEX_CALLBACK_SECRET` through your local secret environment and run:

```sh
sentriex callbacks verify --body callback.json --headers headers.json --json
```

Use `--signature-only` only for inspecting old captured deliveries, not for a live callback receiver.

## Durable processing and retries

Authenticate first. In one application transaction, claim `event_id`, validate the expected application payment/order correlation, and durably record or apply the event. A valid duplicate must not apply financial effects twice. Acknowledge with 2xx after durable acceptance, including already accepted duplicates. If acceptance fails, return a non-2xx result so delivery can retry.

Retries retain the same `event_id` and identical raw payload and generate a fresh timestamp/signature. Non-2xx responses, timeouts, and network failures are retried with backoff; attempts are bounded (currently 10). Callback failure does not undo a credited deposit or completed withdrawal. Monitor failed/exhausted deliveries through merchant settings; the Public API CLI cannot force a server replay.

Deposits emit a credited event after accounting, not on creation/detection or lazy expiry. Withdrawal failures discovered synchronously during create are returned directly and do not produce a failed callback. An asynchronous `withdrawal_order.failed` may report reversal pending; a later reversal does not emit a second failed callback. Retrieve the withdrawal if you need its later reversal state.
