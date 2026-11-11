---
label: Reference quote
icon: graph
order: 45
---

# Reference quote

:::code-tab GET
```
GET /api/v1/cross-border/partner/rates
```
:::

Returns an **indicative** price: the BRL the payer would pay for the payee to
receive a given amount. Nothing is stored or locked, and the price may differ
from a later firm quote. No KYB is needed — use it for pre-calculation.

## Query parameters

| Parameter         | Type   | Required | Description |
| ----------------- | ------ | -------- | ----------- |
| `payout_amount`   | string | yes | Amount the payee receives, in `payout_currency`. Plain decimal string (digits with an optional fractional part, e.g. `"1000.00"`), `> 0`, at most 2 decimal places, below `100000`, at most 32 characters |
| `payout_currency` | string | yes | ISO 4217 code, exact upper case. Supported: `USD` |
| `payer_id`        | string | no  | A payer created with your key. Adds the payer-limit check to the answer (never refuses the quote) |

`payout_currency` is checked first, then `payout_amount`, then `payer_id`.

## Example

```bash
curl "https://staging-payment.velaro.network/api/v1/cross-border/partner/rates?payout_amount=1000&payout_currency=USD" \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field                  | Type    | Description |
| ---------------------- | ------- | ----------- |
| `amount_to_receive`    | string  | Amount the payee receives, in `payout_currency`, 2 decimals |
| `payout_currency`      | string  | Echoed currency |
| `brl_to_pay`           | string  | BRL the payer pays (Velaro's price, before your markup), 2 decimals, rounded up. Includes `iof_amount_brl` when the payer bears the IOF |
| `effective_rate`       | string  | BRL per one unit of `payout_currency` (`brl_to_pay / amount_to_receive`), 6 decimals. Informational |
| `iof_rate`             | string  | IOF tax rate applied, as a fraction (`"0"` when none; `"0.0038"` = 0.38%) |
| `iof_amount_brl`       | string  | IOF in BRL, 2 decimals (`"0.00"` when none) |
| `indicative`           | boolean | Always `true` |
| `quoted_at`            | string  | When the price was produced (RFC 3339, UTC) |
| `payer_limit_exceeded` | boolean | Only with `payer_id`: whether an order of this amount would be refused for that payer's own limits right now. Absent if the limits could not be checked |
| `payer_limit`          | string  | Only when `payer_limit_exceeded` is `true`: the limit — `per_order` or `monthly` |

```json
{
  "amount_to_receive": "1000.00",
  "payout_currency": "USD",
  "brl_to_pay": "5573.39",
  "effective_rate": "5.573390",
  "iof_rate": "0",
  "iof_amount_brl": "0.00",
  "indicative": true,
  "quoted_at": "2026-11-11T12:00:00Z"
}
```

With `payer_id`, when the amount is over that payer's limit:

```json
{
  "amount_to_receive": "1000.00",
  "payout_currency": "USD",
  "brl_to_pay": "5573.39",
  "effective_rate": "5.573390",
  "iof_rate": "0",
  "iof_amount_brl": "0.00",
  "indicative": true,
  "quoted_at": "2026-11-11T12:00:00Z",
  "payer_limit_exceeded": true,
  "payer_limit": "monthly"
}
```

The response carries `Cache-Control: no-store`.

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `unsupported_payout_currency` | `payout_currency` missing, not upper case, or not supported |
| `400` | `invalid_request` | `payout_amount` missing, not a decimal, `≤ 0`, more than 2 decimal places, or longer than 32 characters |
| `400` | `amount_over_limit` | `payout_amount` is `100000` or more |
| `404` | `not_found` | `payer_id` unknown or created with another key |
| `503` | `quote_unavailable` | No price available right now — retry shortly |
| `500` | `internal_error` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
