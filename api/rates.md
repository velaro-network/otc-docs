---
label: Best rates
icon: graph
order: 30
---

# Best rates

:::code-tab GET
```
GET /v1/rates
```
:::

Returns the best competed **buy** and **sell** rate for each supported pair, in
fiat per asset.

## Example

```bash
curl https://otc-api.velaro.network/v1/rates \
  -H "Authorization: Bearer vk_live_your_key"
```

## Response `200`

| Field            | Type   | Description |
| ---------------- | ------ | ----------- |
| `rates`          | array  | One entry per supported pair |
| `rates[].asset`  | string | Stablecoin |
| `rates[].fiat`   | string | Fiat currency |
| `rates[].buy`    | string | Best price to buy the asset (fiat per asset) |
| `rates[].sell`   | string | Best price to sell the asset (fiat per asset) |
| `as_of`          | string | Timestamp of the rates (UTC, RFC 3339) |

```json
{
  "rates": [
    { "asset": "USDT", "fiat": "BRL", "buy": "5.057342", "sell": "5.0012" },
    { "asset": "USDC", "fiat": "BRL", "buy": "5.061", "sell": "5.0039" }
  ],
  "as_of": "2026-10-08T12:00:00Z"
}
```

## Errors

See [Errors](../errors.md) for the full list. This endpoint can return
`401`, `403`, `429` and `503`.
