---
label: Limits
icon: hourglass
order: 70
---

# Limits

## Rate limit

Each API key may make about **20 requests per second**, with bursts of up to
**40**. Exceeding it returns **`429 Too Many Requests`**:

```json
{ "status": 429, "title": "Too Many Requests", "code": "rate_limited", "detail": "too many requests" }
```

No `Retry-After` header is sent; back off briefly and retry. If you need a
higher limit, contact your account manager.

## Request body size

| Endpoint | Max body |
| -------- | -------- |
| `POST /customers` | 120,000,000 bytes |
| `POST /orders` | 16,000,000 bytes |
| `POST /quotes`, `POST /payees` | 64,000 bytes |

A larger body is refused with **`413 payload_too_large`** before it is
processed.

## Documents

Documents (KYB documents and the transaction document) are sent inside the
JSON body, base64-encoded (standard alphabet, with padding).

| Rule | Limit |
| ---- | ----- |
| Size of one document (decoded) | 10,000,000 bytes (10 MB) |
| File type | PDF, JPEG or PNG — detected from the file's bytes, not from the filename or `content_type` |
| Filename | 1–255 characters (path components are stripped) |
| Documents per KYB submission | 40 |
| Total size of one KYB submission (decoded) | 83,886,080 bytes (80 MiB) |
| Documents per payer, all submissions together | 1,000 |
| KYB `profile` object | 65,536 bytes (64 KiB) as JSON |

## KYB uploads in progress

Each Velaro server handles only a few `POST /customers` uploads at the same
time. An upload that arrives while all slots are busy is refused at once —
nothing from it is stored — with **`503 service_busy`** and a `Retry-After`
header (seconds, currently `10`):

```json
{
  "status": 503,
  "title": "Service Unavailable",
  "code": "service_busy",
  "detail": "too many uploads in progress; retry after the Retry-After interval"
}
```

Send the same request again after the `Retry-After` interval (add some jitter
when retrying several uploads). `GET /customers/{id}/status` is never refused
this way.

## Amounts per order

| Rule | Limit |
| ---- | ----- |
| Amount to receive (`payout_amount`) | Greater than 0 and **below USD 100,000** (exclusive: `99999.99` is accepted, `100000` is not) |
| Decimal places in `payout_amount` | At most 2 |
| Payout currency | `USD` |
| Amount strings | At most 32 characters |

The USD 100,000 cap is checked on every quote and again when the order is
created (`400 amount_over_limit`).

### Payer limits

Velaro may also set limits for an individual payer:

- a **per-order** limit — the most one order may be;
- a **monthly** limit — the most the payer's orders may total in one calendar
  month (Brazil time, America/Sao_Paulo). Cancelled, expired and refunded
  orders do not count; an order being refunded counts until the refund
  completes.

Both are in USD of the amount to receive, and inclusive (an order of exactly
the limit is accepted). They are checked on a firm quote that names the payer
(`payer_id`) and again on order creation, and answered with
`422 payer_limit_exceeded`. A reference quote ([`GET /rates`](api/rates.md))
with `payer_id` is never refused for them; it only reports
`payer_limit_exceeded`.

## Firm quote validity

A firm quote can back an order until its `expires_at` (inclusive), currently
**30 minutes** after creation. Always use `expires_at` rather than assuming the
duration.
