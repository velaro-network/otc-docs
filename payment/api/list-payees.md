---
label: List payees
icon: list-unordered
order: 38
---

# List payees

:::code-tab GET
```
GET /api/v1/payees?payer_id={payer_id}
```
:::

Lists one payer's payees, newest first. The full account number is never
returned, only its last 4 digits.

## Query parameters

| Parameter  | Type    | Required | Description |
| ---------- | ------- | -------- | ----------- |
| `payer_id` | string  | yes | Payer id returned by [Create a customer](create-customer.md) |
| `limit`    | integer | no  | Page size, `1`–`200`. Default `50`; larger values are capped at `200` |
| `cursor`   | string  | no  | `next_cursor` from the previous page |

Pagination works as in [List customers](list-customers.md#pagination).

## Example

```bash
curl "https://staging.payment.velaro.network/api/v1/payees?payer_id=payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b" \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field         | Type   | Description |
| ------------- | ------ | ----------- |
| `payees`      | array  | The payees on this page (below) |
| `next_cursor` | string \| null | Cursor for the next page; `null` on the last page |

Each payee (bank fields are its current bank account):

| Field                 | Type   | Description |
| --------------------- | ------ | ----------- |
| `id`                  | string | Payee id |
| `payer_id`            | string | The payer it belongs to |
| `relationship`        | string | As sent to [Register a payee](create-payee.md) |
| `account_holder_name` | string | Account holder name |
| `bank_name`           | string | Bank name |
| `account_last4`       | string | Last 4 digits of the account number |
| `registration_status` | string | Bank account registration status, as [Register a payee](create-payee.md) returns it |
| `created_at`          | string | When the payee was created |

```json
{
  "payees": [
    {
      "id": "payee_3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f",
      "payer_id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
      "relationship": "supplier",
      "account_holder_name": "Supplier Co",
      "bank_name": "Example Bank",
      "account_last4": "6789",
      "registration_status": "registered",
      "created_at": "2026-11-02T09:15:00Z"
    }
  ],
  "next_cursor": null
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `invalid_request` | `payer_id` missing, `limit` not a positive integer, or a `cursor` this endpoint did not return |
| `404` | `not_found` | Unknown `payer_id`, or a payer that is not yours |
| `500` | `list_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
