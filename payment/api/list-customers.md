---
label: List customers
icon: list-unordered
order: 39
---

# List customers

:::code-tab GET
```
GET /api/v1/customers
```
:::

Lists your payers, newest first. Use it to find a payer id you no longer
have: creating a payer again with the same CNPJ is refused with `409`.

## Query parameters

| Parameter | Type    | Required | Description |
| --------- | ------- | -------- | ----------- |
| `cnpj`    | string  | no | Only the payer with this CNPJ. Formatting (`.`, `/`, `-`) is ignored, as on [Create a customer](create-customer.md) |
| `status`  | string  | no | Only payers in this review status: `kyb_pending`, `approved`, `more_docs_needed` or `kyb_rejected` (see [Customer status](customer-status.md)) |
| `limit`   | integer | no | Page size, `1`–`200`. Default `50`; larger values are capped at `200` |
| `cursor`  | string  | no | `next_cursor` from the previous page |

## Pagination

Results are ordered newest first. When `next_cursor` is not `null`, pass it
back as `cursor` (with the same filters) to get the next page. Treat the
cursor as an opaque token: do not build or change it.

## Example

```bash
curl "https://staging.payment.velaro.network/api/v1/customers?cnpj=11.222.333/0001-81" \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field         | Type   | Description |
| ------------- | ------ | ----------- |
| `customers`   | array  | The payers on this page (below); empty when none match |
| `next_cursor` | string \| null | Cursor for the next page; `null` on the last page |

Each payer:

| Field        | Type   | Description |
| ------------ | ------ | ----------- |
| `id`         | string | Payer id |
| `legal_name` | string | Legal name |
| `cnpj`       | string | CNPJ, without formatting |
| `status`     | string | Review status, as [Customer status](customer-status.md) returns it |
| `created_at` | string | When the payer was created |

```json
{
  "customers": [
    {
      "id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
      "legal_name": "Importadora Exemplo Ltda",
      "cnpj": "11222333000181",
      "status": "approved",
      "created_at": "2026-11-01T14:03:22Z"
    }
  ],
  "next_cursor": null
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `invalid_request` | `cnpj` is not a valid CNPJ, unknown `status`, `limit` not a positive integer, or a `cursor` this endpoint did not return |
| `500` | `list_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
