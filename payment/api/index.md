---
label: API Reference
icon: code
order: 50
---

# API Reference

Base URL: `https://staging.payment.velaro.network/api/v1`
(Sandbox — see [Overview](../overview.md#environments)).

All endpoints require authentication — send your key as `X-API-Key: vcb_…`
(see [Authentication](../authentication.md)).

| Method | Path | Description |
| ------ | ---- | ----------- |
| `POST` | [`/customers`](create-customer.md) | Create a payer and submit its KYB documents, or resubmit documents |
| `GET`  | [`/customers`](list-customers.md) | List your payers |
| `GET`  | [`/customers/{id}/status`](customer-status.md) | A payer's KYB review status |
| `POST` | [`/payees`](create-payee.md) | Register a payee and its bank account |
| `GET`  | [`/payees`](list-payees.md) | List a payer's payees |
| `GET`  | [`/rates`](rates.md) | Indicative (reference) quote |
| `POST` | [`/quotes`](create-quote.md) | Create a firm quote |
| `GET`  | [`/quotes/{id}`](get-quote.md) | Look up a firm quote |
| `POST` | [`/orders`](create-order.md) | Create a payment order from a firm quote |
| `GET`  | [`/orders`](list-orders.md) | List your orders |
| `GET`  | [`/orders/{id}/status`](order-status.md) | An order's status |
| `POST` | [`/orders/{id}/cancel`](cancel-order.md) | Cancel an order before payment |

Amounts are **decimal strings**; times are RFC 3339 in UTC.
