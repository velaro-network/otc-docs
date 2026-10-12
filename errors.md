---
label: Errors
icon: alert
order: 60
---

# Errors

Errors are returned as JSON with an `error` message and the matching HTTP status.

```json
{ "error": "side must be BUY or SELL" }
```

| Status | Meaning | Typical cause |
| ------ | ------- | ------------- |
| `400`  | Bad request | Unsupported pair, bad `side`, non-positive `amount`, or invalid JSON |
| `401`  | Unauthorized | Missing or invalid API key |
| `403`  | Forbidden | API key revoked |
| `429`  | Too many requests | More than one request per 5s for the key (see [Rate limits](rate-limits.md)) |
| `503`  | Service unavailable | No price available right now — retry shortly |

A `200` response always contains a full quote (or the rates list).
