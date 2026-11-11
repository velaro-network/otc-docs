---
label: Get a firm quote
icon: search
order: 43
---

# Get a firm quote

:::code-tab GET
```
GET /api/v1/cross-border/heyship/quotes/{id}
```
:::

Returns a firm quote created with your key, while it is still valid.

## Path parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| `id`      | string | The `quote_id` returned by [Create a firm quote](create-quote.md) |

## Example

```bash
curl https://staging-payment.velaro.network/api/v1/cross-border/heyship/quotes/q_3f9c2b7e8a1d4c6f9e0b1a2c3d4e5f60 \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

The same fields as the creation response:

| Field               | Type   | Description |
| ------------------- | ------ | ----------- |
| `quote_id`          | string | Quote id |
| `amount_to_receive` | string | Amount the payee receives, in `payout_currency`, 2 decimals |
| `payout_currency`   | string | Payout currency |
| `brl_to_pay`        | string | BRL Velaro charges, 2 decimals |
| `effective_rate`    | string | BRL per one unit of `payout_currency`, 6 decimals |
| `iof_rate`          | string | IOF tax rate applied, as a fraction |
| `iof_amount_brl`    | string | IOF in BRL, 2 decimals |
| `expires_at`        | string | Last moment the quote can back an order (RFC 3339, UTC) |

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

The response carries `Cache-Control: no-store`.

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `404` | `not_found` | Unknown or malformed id, or a quote created with another key |
| `410` | `quote_expired` | `expires_at` has passed — request a new quote |
| `500` | `internal_error` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
