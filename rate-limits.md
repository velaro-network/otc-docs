---
label: Rate limits
icon: hourglass
order: 70
---

# Rate limits

Each API key may make **one request every 5 seconds**. Prices move slowly, so a
single fresh quote every few seconds is enough for most integrations.

Exceeding the limit returns **`429 Too Many Requests`** with a `Retry-After: 5`
header:

```json
{ "error": "rate limit exceeded, retry in 5s" }
```

Back off and retry after the interval. If you need a higher limit, contact your
account manager.
