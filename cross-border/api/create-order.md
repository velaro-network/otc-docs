---
label: Create an order
icon: package
order: 42
---

# Create an order

:::code-tab POST
```
POST /api/v1/cross-border/partner/orders
```
:::

Creates a payment order from a [firm quote](create-quote.md) and a
[payee](create-payee.md). The payee's payer must be **approved**
([Customer status](customer-status.md)) and the payee's bank account
**registered** (`sphere_status` = `registered`).

A firm quote backs at most one order.

## Request body

| Field                 | Type   | Required | Description |
| --------------------- | ------ | -------- | ----------- |
| `quote_id`            | string | yes | A firm quote created with your key and not expired |
| `payee_id`            | string | yes | A payee created with your key |
| `payment_reason`      | string | yes | One of the payment reasons below |
| `payment_description` | string | yes | 1–500 characters, e.g. the invoice number |
| `brl_all`             | string | yes | Total BRL the payer pays: the quote's `brl_to_pay` plus `markup`. Decimal string, `> 0` |
| `markup`      | string | no  | Your markup in BRL, `≥ 0` and `≤ brl_all`. Omitted means `0` |
| `document`            | object | see below | The transaction document (invoice, contract, …). **Required unless the payee's `relationship` is `self`** |
| `document.filename`       | string | yes | File name, 1–255 characters |
| `document.content_type`   | string | no  | Ignored: the type is detected from the file's bytes |
| `document.content_base64` | string | yes | File content, standard base64. PDF, JPEG or PNG, at most 10,000,000 bytes decoded |

**Amount format.** `brl_all` and `markup` must be plain decimals:
digits with an optional fractional part — no sign, exponent, leading zero
(except `0` itself, e.g. `0.50`) or spaces, at most 32 characters. `5603.39`
is valid; `+5603.39`, `5.60339e3` and `05603.39` are not.

**Price check.** `brl_all − markup` must equal the quote's `brl_to_pay`
exactly; otherwise `400 invalid_request`.

`payment_reason` values: `personal`, `investment`, `real_estate`, `tax`,
`loan`, `bills`, `reimbursement`, `professional_services`, `family_support`,
`education`, `rent`, `donation`, `gift`, `insurance`, `medical`, `savings`,
`travel`, `mortgage`, `fine`, `dividend`, `agriculture`, `import_export`,
`art`, `other`.

```json
{
  "quote_id": "q_3f9c2b7e8a1d4c6f9e0b1a2c3d4e5f60",
  "payee_id": "payee_5d2c7a1b9e8f4a3c2b1d0e9f8a7b6c5d",
  "payment_reason": "import_export",
  "payment_description": "INV-0001",
  "brl_all": "5603.39",
  "markup": "30.00",
  "document": {
    "filename": "invoice-0001.pdf",
    "content_type": "application/pdf",
    "content_base64": "JVBERi0xLjQK..."
  }
}
```

## Example

```bash
curl -X POST https://staging-payment.velaro.network/api/v1/cross-border/partner/orders \
  -H "X-API-Key: vcb_your_key" \
  -H "Content-Type: application/json" \
  -d @order.json
```

## Response `201`

| Field    | Type   | Description |
| -------- | ------ | ----------- |
| `id`     | string | Order id (`order_` + 32 hex characters) |
| `status` | string | The order's status — see [Overview](../overview.md#order-statuses). `quoted`, or `awaiting_pix` when the Pix payment has been set up at creation |
| `pix`    | object | Where the payer pays. Present only while the order is `awaiting_pix` and payment details are available |
| `pix.amount`         | string | BRL to pay (the order's `brl_all`) |
| `pix.expires_at`     | string | Payment deadline (RFC 3339, UTC; may carry fractional seconds). Unpaid by then, the order becomes `pix_expired` |
| `pix.pix_key`        | string | Pix key |
| `pix.qr_payload`     | string | Pix copy-and-paste / QR code payload |
| `pix.qr_image_url`   | string | URL of the QR code image |
| `pix.bank`           | string | Receiving bank |
| `pix.branch`         | string | Receiving branch |
| `pix.account_number` | string | Receiving account number |
| `pix.holder_name`    | string | Receiving account holder |
| `pix.holder_tax_id`  | string | Receiving account holder's tax id |

Every `pix` field is omitted when it has no value.

```json
{
  "id": "order_9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d",
  "status": "awaiting_pix",
  "pix": {
    "pix_key": "pagamentos@example.com.br",
    "bank": "Example Bank",
    "branch": "0001",
    "account_number": "123456-7",
    "holder_name": "Example Pagamentos Ltda",
    "holder_tax_id": "11222333000181",
    "amount": "5603.39",
    "expires_at": "2026-11-12T12:00:00Z",
    "qr_payload": "00020126580014br.gov.bcb.pix..."
  }
}
```

Follow the order with [Order status](order-status.md) or the
`order.status_changed` [webhook](../webhooks.md).

## Retries

There is no idempotency key; the **`quote_id`** plays that role. Sending a
request again with a `quote_id` that already backs one of your orders:

- with the same `payee_id`, `brl_all`, `markup`, `payment_reason`
  and `payment_description` (compared as exact strings — `"30.00"` and `"30"`
  differ) returns the **existing** order with `200` and its current status.
  The `document` is not compared and need not be sent again. Expiry of the
  quote or a later change of the payer's review does not affect this;
- with any of those fields different, returns `409 quote_already_used`.

So a request that timed out can safely be sent again unchanged.

## Errors

Checks run roughly in this order: retry match, quote, payee, payer and payee
readiness, body fields and document format, payer limits, document upload, payment set-up.

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `bad_request` | Body is not valid JSON or a field has the wrong JSON type |
| `400` | `invalid_request` | A field fails its rule: `brl_all` missing or malformed, `markup` negative, malformed or above `brl_all`, `brl_all − markup` ≠ the quote's `brl_to_pay`, unknown `payment_reason`, `payment_description` empty or over 500 characters, `document` missing for a non-`self` payee, document not valid base64, over 10 MB, not PDF/JPEG/PNG, or with a missing/too-long filename |
| `400` | `amount_over_limit` | The quote's amount is no longer below the per-order cap |
| `404` | `not_found` | Quote, payee or payer unknown or created with another key (the `detail` says which) |
| `409` | `quote_already_used` | The quote already backs an order with different fields (see [Retries](#retries)) |
| `410` | `quote_expired` | The quote's `expires_at` has passed — create a new quote |
| `413` | `payload_too_large` | Body over 16,000,000 bytes |
| `422` | `payer_not_approved` | The payer's KYB is not approved |
| `422` | `payee_not_registered` | The payee's bank account is not registered for payouts yet (`sphere_status` not `registered`) |
| `422` | `payout_provider_not_supported` | Payouts for this payer cannot be made in this environment — contact support |
| `422` | `unsupported_payout_currency` | The quote's payout currency cannot be paid out for this payer |
| `422` | `payer_limit_exceeded` | Over the payer's per-order or monthly limit. Same extra members as on [Create a firm quote](create-quote.md#payer_limit_exceeded) |
| `422` | `transaction_document_rejected` | The transaction document was refused. Carries `sphere_status` (number, the HTTP status of the refusal) and `sphere_reason` (string, the reason given, may be empty). Fix the document and send the request again |
| `422` | `order_not_accepted` | The order could not be accepted at this time |
| `503` | `otc_unavailable` | The Pix payment could not be set up — temporary. Nothing was created; send the same request again |
| `500` | `order_lookup_failed`, `quote_lookup_failed`, `payee_lookup_failed`, `payer_lookup_failed`, `create_order_failed` | Unexpected failure |

For every `4xx` except `409`, and for `503`, no order was created and the
quote is still free (until it expires). After a `5xx`, send the same request
again: if the order was in fact created, the retry returns it.

Plus the codes common to every endpoint — see [Errors](../errors.md).
