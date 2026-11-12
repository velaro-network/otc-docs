---
label: Order status
icon: pulse
order: 41
---

# Order status

:::code-tab GET
```
GET /api/v1/orders/{id}/status
```
:::

Returns an order's current status. The same value is pushed by the
`order.status_changed` [webhook](../webhooks.md); this endpoint is the
authoritative source.

The status only moves **forward**: a value once returned is never followed by
an earlier one, and the first final status (`settled`, `pix_expired`,
`cancelled` or `refunded`) is final. See [Overview](../overview.md#order-statuses) for every status.

## Path parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| `id`      | string | Order id returned by [Create an order](create-order.md) |

## Example

```bash
curl https://staging.payment.velaro.network/api/v1/orders/order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d/status \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field      | Type   | Description |
| ---------- | ------ | ----------- |
| `order_id` | string | Order id |
| `status`   | string | `quoted`, `awaiting_pix`, `paid`, `converted`, `settled`, `failed`, `pix_expired`, `cancelled` or `refunded` |
| `reason`   | string | Only with `failed`: `refund_pending` (the payment will be refunded to the payer) |
| `uetr`     | string \| null | Reserved for the SWIFT tracking reference (UETR) of the payout. Not provided yet: currently left out of the response; treat it as optional (absent or `null`) |

```json
{
  "order_id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
  "status": "settled"
}
```

```json
{
  "order_id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
  "status": "failed",
  "reason": "refund_pending"
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `404` | `not_found` | Unknown id, or an order created with another key |
| `404` | `no_status` | The order has no status to report yet |
| `500` | `get_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
