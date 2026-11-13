---
label: Overview
icon: book
order: 90
---

# Overview

The **Velaro Payment API** moves a payment from a Brazilian business
(the **payer**, your customer) to its counterparty abroad (the **payee**).
The payer pays BRL by Pix; the payee receives the quoted amount in USD.

Every endpoint sits under one base URL and uses one API key (see [Authentication](authentication.md)).

## Environments

| Environment | Base URL |
| ----------- | -------- |
| Sandbox (staging) | `https://staging.payment.velaro.network/api/v1` |

Full endpoint URL = base URL + path, for example:

```
https://staging.payment.velaro.network/api/v1/rates
```

## End-to-end flow

1. **Onboard the payer (KYB).** [`POST /customers`](api/create-customer.md)
   with the company's details and documents. Velaro reviews it; follow the
   result with [`GET /customers/{id}/status`](api/customer-status.md) or the
   `payer.review_result` [webhook](webhooks.md). If documents are missing,
   resubmit with the same endpoint.
2. **Register the payee.** [`POST /payees`](api/create-payee.md) with the
   payee's bank account. A payee is reused for all of that payer's later
   orders.
3. **Price the payment.** [`GET /rates`](api/rates.md) for an indicative
   price at any time (no KYB needed), then [`POST /quotes`](api/create-quote.md)
   for a **firm quote**, valid until its `expires_at`.
4. **Create the order.** [`POST /orders`](api/create-order.md) from the firm
   quote and the payee, with your markup, the payment reason and the
   transaction document. The payer must be approved; the payee's bank account
   must be registered when the payer's payouts need it up front.
5. **Payer pays by Pix.** The order moves to `awaiting_pix`; the creation
   answer carries the Pix payment details (`pix`) once they are available.
6. **Velaro converts and pays out.** The order moves through `paid`,
   `converted` and `settled`.
7. **Follow the status.** Poll [`GET /orders/{id}/status`](api/order-status.md)
   or receive `order.status_changed` [webhooks](webhooks.md).

## Order statuses

The status you see is always one of these values. It only ever moves
**forward**: once a status has been reported, you never see an earlier one,
and the first final status reported is final.

| Status | Meaning |
| ------ | ------- |
| `quoted` | Order created from a firm quote; payment instructions not issued yet |
| `awaiting_pix` | Waiting for the payer's Pix payment |
| `paid` | Payment received; conversion and payout are being prepared |
| `converted` | Funds have been sent on for payout to the payee |
| `settled` | Paid out to the payee. `uetr` (SWIFT tracking reference) is not provided yet: the field is reserved and currently left out. **Final** |
| `failed` | The payment could not be completed and will be refunded to the payer. Always with `reason: "refund_pending"` |
| `pix_expired` | The Pix payment deadline passed without payment; the order is closed. **Final**. Reported by the status endpoint only — no webhook is sent for it |
| `cancelled` | The order was cancelled before payment — by you (see [Cancel an order](api/cancel-order.md)) or by Velaro operations. **Final** |

Notes:

- While Velaro handles an issue on an order internally, you keep seeing the
  status it had before.
- After a refund completes, the order keeps reporting `failed`.
- An order cancelled before payment reports `cancelled`, and an
  `order.status_changed` webhook is sent for it. (An expired Pix is reported
  as `pix_expired` instead.) See [Cancel an order](api/cancel-order.md).

## Retrying requests

There are no idempotency keys. Where a retry is safe:

- `POST /orders` is idempotent **per quote**: a retry with the same `quote_id`
  and the same fields returns the original order (`200`) instead of creating
  a second one. See [Create an order](api/create-order.md#retries).
- `POST /payees` is **not** idempotent: every successful call creates a new
  payee.
- `POST /customers` creates a payer once per CNPJ; a second payer with the
  same CNPJ is refused with `409`.
- `POST /quotes` creates a new quote on every call.

## Conventions

- Bodies are JSON (`Content-Type: application/json`).
- Amounts are **decimal strings** (`"1000.00"`), never JSON numbers.
- Times are RFC 3339 in **UTC** (`"2026-11-11T12:00:00Z"`).
- Ids are opaque strings with a type prefix: `payer_…`, `payee_…`, `q_…`,
  `order_…`.
- Every response carries an `X-Request-Id` header. Quote it when you
  contact [support](support.md).
