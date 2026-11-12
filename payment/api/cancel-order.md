---
label: Cancel order
icon: x-circle
order: 40
---

# Cancel an order

:::code-tab POST
```
POST /api/v1/orders/{id}/cancel
```
:::

Cancels an order that has **not been paid yet** (it is still `awaiting_pix`).
Velaro first confirms with its settlement desk that no payment has landed on
the order, cancels the linked trade, then moves the order to `cancelled`. The
status endpoint then returns `cancelled` and an `order.status_changed`
[webhook](../webhooks.md) is sent.

You can only cancel your **own** orders, and only before a payment is
credited. Once a payment has been credited you cannot cancel — ask Velaro to
refund it instead (see the error below).

!!!warning Stop showing the Pix QR after cancelling
Cancelling does not disable the Pix QR code the payer was given. If the payer
pays it after you cancel, the payment arrives with no order to attribute it to
and Velaro has to refund it by hand. Stop presenting the QR to the payer as
soon as you cancel.
!!!

## Path parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| `id`      | string | Order id returned by [Create an order](create-order.md) |

## Body (optional)

| Field  | Type   | Description |
| ------ | ------ | ----------- |
| `note` | string | Optional free-text reason, recorded on the order |

## Example

```bash
curl -X POST https://staging.payment.velaro.network/api/v1/orders/order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d/cancel \
  -H "X-API-Key: vcb_your_key" \
  -H "Content-Type: application/json" \
  -d '{"note": "customer changed their mind"}'
```

## Response `200`

| Field      | Type   | Description |
| ---------- | ------ | ----------- |
| `order_id` | string | Order id |
| `status`   | string | Always `cancelled` |

```json
{
  "order_id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
  "status": "cancelled"
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `404` | `not_found` | Unknown id, or an order created with another key |
| `409` | `order_already_credited` | A payment has already been credited to the order. Cancelling is no longer possible — ask Velaro to refund the payer instead |
| `409` | `order_wrong_status` | The order is past `awaiting_pix` (already paid, converting, settled, …) |
| `400` | `bad_request` | Malformed JSON body |

Plus the codes common to every endpoint — see [Errors](../errors.md).
