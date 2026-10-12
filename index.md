---
label: Home
icon: home
order: 100
---

# Velaro OTC API

Competitive **BRL ↔ stablecoin** quotes over a simple REST API.

Every quote is priced across all of Velaro's liquidity sources and returned as a
single **best price** — you never see or choose the underlying provider. The API
is **quote-only**: it returns indicative prices, it does not execute trades or
move funds.

[!ref Get started](overview.md)
[!ref API Reference](api/quote.md)

---

:::card
**Best price, one call** — each quote is competed across all of Velaro's liquidity sources; you get the single best rate.
:::

:::card
**Simple & authenticated** — a REST endpoint and a Bearer API key. No signing, no SDK required.
:::

:::card
**Quote-only** — indicative prices only. No trade execution, no fund movement.
:::

:::card
**BRL ↔ stablecoin** — USDT/BRL and USDC/BRL, both directions.
:::
