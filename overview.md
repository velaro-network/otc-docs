---
label: Overview
icon: book
order: 90
---

# Overview

The **Velaro OTC Quote API** returns indicative BRL ↔ stablecoin prices. Every
quote is priced competitively across Velaro's liquidity sources and returned as
a single **best price** — you never see or choose the underlying provider.

The API is **quote-only**: it returns prices, it does not execute trades or move
funds. Prices are indicative and may change before settlement.

## Base URL

```
https://otc-api.velaro.network
```

## Supported pairs

| Asset | Fiat | Directions |
| ----- | ---- | ---------- |
| USDT  | BRL  | BUY · SELL |
| USDC  | BRL  | BUY · SELL |

`BUY` = buy the asset with fiat. `SELL` = sell the asset for fiat.

## Quickstart

Request a quote to buy 100 USDT with BRL:

```bash
curl -X POST https://otc-api.velaro.network/v1/quote \
  -H "Authorization: Bearer vk_live_your_key" \
  -H "Content-Type: application/json" \
  -d '{"asset":"USDT","fiat":"BRL","side":"BUY","amount":"100"}'
```

```json
{
  "asset": "USDT",
  "fiat": "BRL",
  "side": "BUY",
  "amount": "100",
  "price": "5.057342",
  "total": "505.7342",
  "quoted_at": "2026-10-08T09:00:00-03:00"
}
```

`price` is the best rate in fiat per asset; `total` is `amount × price`.

See the full [API Reference](api/quote.md) for every field and endpoint.
