---
label: Rate limits
icon: hourglass
order: 70
---

# Rate limits

Each API key may make **up to two requests per second**.

Exceeding the limit returns **`429 Too Many Requests`** with a `Retry-After: 1`
header:

```json
{ "error": "rate limit exceeded, retry in 1s" }
```

Back off and retry after a second. If you need a higher limit, contact your
account manager.
