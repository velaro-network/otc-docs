---
label: Errors
icon: alert
order: 60
---

# Errors

Every error is a JSON object in the problem+json shape (loosely RFC 7807),
with the matching HTTP status. It is sent with
`Content-Type: application/json`.

```json
{
  "status": 404,
  "title": "Not Found",
  "code": "not_found",
  "detail": "quote not found"
}
```

| Field    | Type    | Description |
| -------- | ------- | ----------- |
| `status` | number  | The HTTP status |
| `title`  | string  | The HTTP status text |
| `code`   | string  | Stable, machine-readable error code. **Branch on this.** |
| `detail` | string  | Human-readable explanation. Wording may change; do not parse it |

Some errors add extra members (for example `field` and `reason` on a payee
validation error, or the limit figures on `payer_limit_exceeded`). They are
documented on the endpoint that returns them.

`500` errors always carry the detail `"internal error"` (`503` details vary — do not parse them); quote the response's
`X-Request-Id` header when you contact [support](support.md).

## Status codes

| Status | Meaning | Typical cause |
| ------ | ------- | ------------- |
| `400` | Bad request | Malformed JSON, a missing or invalid field |
| `401` | Unauthorized | Missing, unknown or revoked API key |
| `403` | Forbidden | Key not issued for this API |
| `404` | Not found | Unknown id, or an id that belongs to another key |
| `409` | Conflict | Duplicate CNPJ, a quote already used by a different order, a concurrent update |
| `410` | Gone | Firm quote expired |
| `413` | Payload too large | Body over the endpoint's limit (see [Limits](limits.md)) |
| `422` | Unprocessable | Well-formed request the current state does not allow (payer not approved, limit exceeded, document rejected, …) |
| `429` | Too many requests | Rate limit (see [Limits](limits.md)) |
| `500` | Internal error | Unexpected failure — retry later or contact support |
| `503` | Service unavailable | Temporary; retry later |

## Codes on every endpoint

| Status | `code` | When |
| ------ | ------ | ---- |
| `401` | `missing_api_key` | No key sent |
| `401` | `invalid_api_key` | Key unknown or revoked |
| `403` | `scope_forbidden` | Key not issued for this API |
| `429` | `rate_limited` | Too many requests for the key |
| `401` | `unauthenticated` | A path or HTTP method that is not documented here, even with a valid key. Check the method and path against the [API reference](api/index.md) |

## Codes by endpoint

| Endpoint | Status | `code` |
| -------- | ------ | ------ |
| [`GET /rates`](api/rates.md) | `400` | `unsupported_payout_currency`, `invalid_request`, `amount_over_limit` |
| | `404` | `not_found` (payer) |
| | `503` | `quote_unavailable` |
| | `500` | `internal_error` |
| [`POST /quotes`](api/create-quote.md) | `400` | `bad_request`, `invalid_request`, `unsupported_payout_currency`, `amount_over_limit` |
| | `404` | `not_found` (payer) |
| | `413` | `payload_too_large` |
| | `422` | `payer_limit_exceeded` |
| | `503` | `quote_unavailable` |
| | `500` | `internal_error` |
| [`GET /quotes/{id}`](api/get-quote.md) | `404` | `not_found` |
| | `410` | `quote_expired` |
| | `500` | `internal_error` |
| [`POST /customers`](api/create-customer.md) | `400` | `bad_request`, `invalid_request` |
| | `404` | `not_found` (resubmission) |
| | `409` | `conflict` |
| | `413` | `payload_too_large` |
| | `422` | `file_unconvertible` |
| | `503` | `service_busy` |
| | `500` | `create_customer_failed` |
| [`GET /customers/{id}/status`](api/customer-status.md) | `404` | `not_found` |
| | `500` | `get_failed` |
| [`POST /payees`](api/create-payee.md) | `400` | `bad_request`, `invalid_request` |
| | `404` | `not_found` (payer) |
| | `409` | `conflict` |
| | `413` | `payload_too_large` |
| | `500` | `create_payee_failed` |
| [`POST /orders`](api/create-order.md) | `400` | `bad_request`, `invalid_request`, `amount_over_limit` |
| | `404` | `not_found` (quote, payee or payer) |
| | `409` | `quote_already_used` |
| | `410` | `quote_expired` |
| | `413` | `payload_too_large` |
| | `422` | `payer_not_approved`, `payee_not_registered`, `payout_provider_not_supported`, `unsupported_payout_currency`, `payer_limit_exceeded`, `transaction_document_rejected`, `file_unconvertible`, `order_not_accepted` |
| | `503` | `otc_unavailable` |
| | `500` | `order_lookup_failed`, `quote_lookup_failed`, `payee_lookup_failed`, `payer_lookup_failed`, `create_order_failed` |
| [`GET /orders/{id}/status`](api/order-status.md) | `404` | `not_found`, `no_status` |
| | `500` | `get_failed` |

Each endpoint page explains when its codes occur.
