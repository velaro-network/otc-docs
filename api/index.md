---
label: API Reference
icon: code
order: 50
---

# API Reference

Base URL: `https://otc.velaro.network`

All endpoints require authentication — send your key as `Authorization: Bearer vk_live_…`
(see [Authentication](../authentication.md)).

| Method | Path | Description |
| ------ | ---- | ----------- |
| `POST` | [`/v1/quote`](quote.md) | Create a best-price quote for a pair, side and amount |
| `GET`  | [`/v1/rates`](rates.md) | Best buy/sell rate for every supported pair |

Prices are **indicative** and in **fiat per asset**. The API is quote-only.
