---
label: Create a quote
icon: arrow-right
order: 40
---

# Create a quote

:::code-tab POST
```
POST /v1/quote
```
:::

Returns the best competed price for a single pair, side and amount. The price is
the best fiat-per-asset rate across all of Velaro's liquidity sources; `total`
is `amount × price` in the fiat currency.

## Request body

| Field    | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| `asset`  | string | yes | Stablecoin — `USDT` or `USDC` |
| `fiat`   | string | yes | Fiat currency — `BRL` |
| `side`   | string | yes | `BUY` (buy the asset with fiat) or `SELL` (sell the asset for fiat) |
| `amount` | string | yes | Amount in asset units, decimal string, must be `> 0` |

```json
{ "asset": "USDT", "fiat": "BRL", "side": "BUY", "amount": "100" }
```

## Example

```bash
curl -X POST https://otc.velaro.network/v1/quote \
  -H "Authorization: Bearer vk_live_your_key" \
  -H "Content-Type: application/json" \
  -d '{"asset":"USDT","fiat":"BRL","side":"BUY","amount":"100"}'
```

## Response `200`

| Field       | Type   | Description |
| ----------- | ------ | ----------- |
| `asset`     | string | Echoed asset |
| `fiat`      | string | Echoed fiat |
| `side`      | string | Echoed side |
| `amount`    | string | Echoed amount |
| `price`     | string | Best price, in fiat per asset |
| `total`     | string | `amount × price`, in fiat |
| `quoted_at` | string | When the quote was produced (UTC, RFC 3339) |

```json
{
  "asset": "USDT",
  "fiat": "BRL",
  "side": "BUY",
  "amount": "100",
  "price": "5.057342",
  "total": "505.7342",
  "quoted_at": "2026-10-08T12:00:00Z"
}
```

## Errors

See [Errors](../errors.md) for the full list. This endpoint can return
`400`, `401`, `403`, `429` and `503`.
