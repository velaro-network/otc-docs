---
label: List orders
icon: list-unordered
order: 37
---

# List orders

:::code-tab GET
```
GET /api/v1/orders
```
:::

Lists your orders, newest first.

## Query parameters

| Parameter      | Type    | Required | Description |
| -------------- | ------- | -------- | ----------- |
| `payer_id`     | string  | no | Only this payer's orders |
| `status`       | string  | no | Only orders in this status: `quoted`, `awaiting_pix`, `paid`, `converted`, `settled`, `failed`, `pix_expired`, `cancelled` or `refunded` (see [Order status](order-status.md)) |
| `created_from` | string  | no | Only orders created at or after this time (RFC 3339, e.g. `2026-11-01T00:00:00Z`) |
| `created_to`   | string  | no | Only orders created before this time (RFC 3339) |
| `limit`        | integer | no | Page size, `1`–`200`. Default `50`; larger values are capped at `200` |
| `cursor`       | string  | no | `next_cursor` from the previous page |

Pagination works as in [List customers](list-customers.md#pagination).

## Example

```bash
curl "https://staging.payment.velaro.network/api/v1/orders?status=awaiting_pix&limit=20" \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field         | Type   | Description |
| ------------- | ------ | ----------- |
| `orders`      | array  | The orders on this page (below) |
| `next_cursor` | string \| null | Cursor for the next page; `null` on the last page |

Each order:

| Field             | Type   | Description |
| ----------------- | ------ | ----------- |
| `id`              | string | Order id |
| `payer_id`        | string | Payer id |
| `payee_id`        | string | Payee id |
| `status`          | string | Status, as [Order status](order-status.md) returns it |
| `brl_all`         | string | Total BRL the payer pays |
| `payout_amount`   | string | Amount the payee receives, in `payout_currency` |
| `payout_currency` | string | ISO 4217 currency of the payout |
| `created_at`      | string | When the order was created |

```json
{
  "orders": [
    {
      "id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
      "payer_id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
      "payee_id": "payee_3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f",
      "status": "awaiting_pix",
      "brl_all": "5603.39",
      "payout_amount": "1000.00",
      "payout_currency": "USD",
      "created_at": "2026-11-03T10:20:00Z"
    }
  ],
  "next_cursor": "YzF8MTc2MjE2NTIwMDAwMDAwMHxvcmRlcl85YThi"
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `invalid_request` | Unknown `status`, `created_from` / `created_to` not RFC 3339, `limit` not a positive integer, or a `cursor` this endpoint did not return |
| `500` | `list_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
