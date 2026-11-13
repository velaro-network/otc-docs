---
label: Create a firm quote
icon: arrow-right
order: 44
---

# Create a firm quote

:::code-tab POST
```
POST /api/v1/quotes
```
:::

Locks Velaro's price for a payout amount until `expires_at`. An order is
created from a firm quote ([Create an order](create-order.md)); the order's
`brl_all` must equal this quote's `brl_to_pay` plus your markup.

The answer is Velaro's own price only — add your markup on top.

## Request body

| Field             | Type   | Required | Description |
| ----------------- | ------ | -------- | ----------- |
| `payout_amount`   | string | yes | Amount the payee receives, in `payout_currency`. Plain decimal string, e.g. `"1000.00"` (a JSON number is refused), `> 0`, at most 2 decimal places, below `100000`, at most 32 characters |
| `payout_currency` | string | yes | ISO 4217 code, exact upper case. Supported: `USD` |
| `payer_id`        | string | no  | The payer the quote is for (created with your key). An amount over that payer's own limits is refused with `422 payer_limit_exceeded` |

```json
{ "payout_amount": "1000", "payout_currency": "USD" }
```

## Example

```bash
curl -X POST https://staging.payment.velaro.network/api/v1/quotes \
  -H "X-API-Key: vcb_your_key" \
  -H "Content-Type: application/json" \
  -d '{"payout_amount":"1000","payout_currency":"USD"}'
```

## Response `201`

| Field               | Type   | Description |
| ------------------- | ------ | ----------- |
| `quote_id`          | string | Quote id (`q_` + 32 hex characters). Pass it to [Create an order](create-order.md) |
| `amount_to_receive` | string | Amount the payee receives, in `payout_currency`, 2 decimals |
| `payout_currency`   | string | Echoed currency |
| `brl_to_pay`        | string | BRL Velaro charges, 2 decimals, rounded up. Includes `iof_amount_brl` when the payer bears the IOF |
| `effective_rate`    | string | BRL per one unit of `payout_currency`, 6 decimals. Informational |
| `iof_rate`          | string | IOF tax rate applied, as a fraction (`"0"` when none) |
| `iof_amount_brl`    | string | IOF in BRL, 2 decimals (`"0.00"` when none) |
| `expires_at`        | string | Last moment the quote can back an order (RFC 3339, UTC). Currently 30 minutes after creation |

```json
{
  "quote_id": "q_3f9c2b7e8a1d4c6f9e0b1a2c3d4e5f60",
  "amount_to_receive": "1000.00",
  "payout_currency": "USD",
  "brl_to_pay": "5573.39",
  "effective_rate": "5.573390",
  "iof_rate": "0",
  "iof_amount_brl": "0.00",
  "expires_at": "2026-11-11T12:30:00Z"
}
```

Every call creates a new quote. The response carries `Cache-Control: no-store`.

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `bad_request` | Body is not valid JSON, or a field has the wrong JSON type |
| `400` | `invalid_request` | `payout_amount` missing, not a decimal, `≤ 0`, more than 2 decimal places, or longer than 32 characters |
| `400` | `unsupported_payout_currency` | `payout_currency` missing, not upper case, or not supported (checked before the amount) |
| `400` | `amount_over_limit` | `payout_amount` is `100000` or more |
| `404` | `not_found` | `payer_id` unknown or created with another key |
| `413` | `payload_too_large` | Body over 64,000 bytes |
| `422` | `payer_limit_exceeded` | With `payer_id`: the amount goes over that payer's per-order or monthly limit. See below |
| `503` | `quote_unavailable` | No price available right now — retry shortly |
| `500` | `internal_error` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).

### `payer_limit_exceeded`

Carries extra members describing the limit (amounts in USD, 2 decimals):

| Field               | Type   | Description |
| ------------------- | ------ | ----------- |
| `limit`             | string | `per_order` or `monthly` |
| `limit_amount`      | string | The limit that was exceeded |
| `amount`            | string | The requested amount |
| `monthly_used`      | string | What the payer's orders already total this month |
| `monthly_remaining` | string | What the month still allows; present only when the payer has a monthly limit |
| `currency`          | string | `USD` |

```json
{
  "status": 422,
  "title": "Unprocessable Entity",
  "code": "payer_limit_exceeded",
  "detail": "amount 1000.00 USD is over the payer's per-order limit of 500.00 USD",
  "limit": "per_order",
  "limit_amount": "500.00",
  "amount": "1000.00",
  "monthly_used": "0.00",
  "currency": "USD"
}
```
