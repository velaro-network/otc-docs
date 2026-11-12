---
label: Webhooks
icon: broadcast
order: 40
---

# Webhooks

Velaro notifies you when an **order's status** changes and when a **payer's
KYB review** reaches a result, so you do not have to poll. The status
endpoints stay authoritative; a webhook carries the same value.

You give Velaro one HTTPS URL; Velaro gives you a shared **signing secret**.
Contact [support](support.md) to set them up.

## Request

Every delivery is a `POST` of a JSON envelope:

```
POST https://your-endpoint.example.com/velaro
Content-Type: application/json
Velaro-Timestamp: 1793275200
Velaro-Signature: 5c0f…(64 lowercase hex characters)
Velaro-Event-Id: hswevt_3f9c2b7e8a1d4c6f9e0b1a2c3d4e5f60
```

```json
{
  "id": "hswevt_3f9c2b7e8a1d4c6f9e0b1a2c3d4e5f60",
  "type": "order.status_changed",
  "sequence": 3,
  "created_at": "2026-11-11T12:00:00.123456Z",
  "data": {
    "order_id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
    "status": "paid"
  }
}
```

### Headers

| Header             | Description |
| ------------------ | ----------- |
| `Velaro-Timestamp` | Unix time in seconds, decimal, taken when **this attempt** is sent (a retry has a new one) |
| `Velaro-Signature` | Lowercase hex `HMAC-SHA256(secret, "{Velaro-Timestamp}.{raw body}")` |
| `Velaro-Event-Id`  | The envelope's `id` |

### Envelope

| Field        | Type   | Description |
| ------------ | ------ | ----------- |
| `id`         | string | Event id (`hswevt_` + 32 hex characters). Identical on every retry — **deduplicate on it** |
| `type`       | string | `order.status_changed` or `payer.review_result` |
| `sequence`   | number | Increases per (`type`, order or payer id), starting at 1. Not necessarily consecutive |
| `created_at` | string | When the change happened, not when this attempt was sent (RFC 3339, UTC, up to microsecond precision; trailing zeros dropped) |
| `data`       | object | The event payload, by `type` (below) |

The body of an event is byte-identical on every retry. Key order and
whitespace are not guaranteed: parse it as JSON.

## Event types

### `order.status_changed`

Sent when an order's status (the value [Order status](api/order-status.md)
returns) moves to a new value.

| Field      | Type   | Description |
| ---------- | ------ | ----------- |
| `order_id` | string | Order id |
| `status`   | string | `quoted`, `awaiting_pix`, `paid`, `converted`, `settled`, `failed` or `cancelled` |
| `reason`   | string | Only with `failed`: `refund_pending` |
| `uetr`     | string \| null | Reserved for the payout's SWIFT tracking reference. Not provided yet: currently left out of the event; treat it as optional (absent or `null`) |

```json
{ "order_id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d", "status": "settled" }
```

- Sent once per **change**, forward only — the same rule as the status
  endpoint. A status you already polled is not pushed again.
- `pix_expired` is **not** pushed: poll [Order status](api/order-status.md)
  for an order still `awaiting_pix` after its `pix.expires_at`.
- `cancelled` **is** pushed: when an order is cancelled before payment, by you
  (see [Cancel an order](api/cancel-order.md)) or by Velaro operations.
- An order created as `quoted` is usually not pushed until it moves on; you
  already have `quoted` from the creation answer.

### `payer.review_result`

Sent when a payer's KYB review reaches a result, or when the documents
requested or the reviewer's note change.

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| `payer_id`     | string | Payer id |
| `result`       | string | `approved`, `more_docs_needed` or `kyb_rejected` — the same value [Customer status](api/customer-status.md) returns as `status` |
| `missing_docs` | array of strings | Only with `more_docs_needed`: the documents still needed |
| `note`         | string | Only with `more_docs_needed` or `kyb_rejected`, when the reviewer wrote one |

```json
{ "payer_id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b", "result": "more_docs_needed", "missing_docs": ["bank_statement"], "note": "bank statement older than 90 days" }
```

- `kyb_pending` (under review) is never pushed. A resubmission therefore
  sends nothing until the next result — which may be `more_docs_needed`
  again.
- A changed `missing_docs` list or `note` with the same `result` is sent
  again.

## Verifying a delivery

1. Read the **raw** request body bytes, `Velaro-Timestamp` and
   `Velaro-Signature`. Do not re-serialize the JSON.
2. Compute `HMAC-SHA256(secret, timestamp + "." + body)` and hex-encode it
   (lowercase).
3. Compare it with `Velaro-Signature` in constant time. Reject on mismatch.
4. Reject a timestamp more than 5 minutes from your clock. Every attempt is
   signed afresh, so a legitimate retry always passes.

Test vector: secret `hs_test_secret`, timestamp `1793275200`, body `{"a":1}` →
`9c6617bf4db25a9d0492d81843b508fbab67e02aeb4d1b5e5aa826e2c8fe34b8`.

```bash
printf '1793275200.{"a":1}' | openssl dgst -sha256 -hmac hs_test_secret
```

+++ Node.js
```js
const crypto = require("crypto");

function verify(secret, rawBody, timestamp, signature) {
  const expected = crypto
    .createHmac("sha256", secret)
    .update(`${timestamp}.`)
    .update(rawBody) // Buffer of the raw request body
    .digest("hex");
  const a = Buffer.from(expected);
  const b = Buffer.from(signature || "");
  if (a.length !== b.length || !crypto.timingSafeEqual(a, b)) return false;
  return Math.abs(Date.now() / 1000 - Number(timestamp)) <= 300;
}
```
+++ Python
```python
import hashlib, hmac, time

def verify(secret: bytes, raw_body: bytes, timestamp: str, signature: str) -> bool:
    expected = hmac.new(secret, timestamp.encode() + b"." + raw_body, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(expected, signature or ""):
        return False
    return abs(time.time() - int(timestamp)) <= 300
```
+++

## Responding

- Answer any **2xx** to acknowledge. The response body is ignored.
- Anything else is "not delivered" and is retried: a non-2xx status, a
  redirect (`3xx` is not followed), no answer within **5 seconds**, or a
  connection error.
- Answer `2xx` for an event `id` you have already processed — duplicates are
  expected. Acknowledge quickly and do slow work afterwards.

## Retries

Delivery is **at least once**. A failed event is retried with the same `id`
and body:

| After failed attempt | Next attempt in |
| -------------------- | --------------- |
| 1 | 10 s |
| 2 | 1 min |
| 3 | 5 min |
| 4 | 15 min |
| 5 | 1 h |
| 6 – 19 | 6 h each |

Times are approximate (up to about 10 s late). After the **20th** failed
attempt — about 3.8 days — the event is dropped; use the status endpoints to
catch up.

## Ordering

Events about the same order (or the same payer) are sent one at a time, in
`sequence` order: a later event is held back until the earlier one is
delivered or dropped. A dropped event leaves a gap in `sequence`; that is
normal.

Your own queue may still process deliveries out of order, so for each order
or payer id **discard a delivery whose `sequence` is not higher than the last
one you accepted**. When in doubt, call the status endpoint.
