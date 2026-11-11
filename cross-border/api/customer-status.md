---
label: Customer status
icon: checklist
order: 47
---

# Customer status

:::code-tab GET
```
GET /api/v1/cross-border/partner/customers/{id}/status
```
:::

Returns a payer's KYB review status. The same information is pushed by the
`payer.review_result` [webhook](../webhooks.md).

## Path parameters

| Parameter | Type   | Description |
| --------- | ------ | ----------- |
| `id`      | string | Payer id returned by [Create a customer](create-customer.md) |

## Example

```bash
curl https://staging-payment.velaro.network/api/v1/cross-border/partner/customers/payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b/status \
  -H "X-API-Key: vcb_your_key"
```

## Response `200`

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| `id`           | string | Payer id |
| `status`       | string | `kyb_pending`, `approved`, `more_docs_needed` or `kyb_rejected` (below) |
| `missing_docs` | array of strings | Only with `more_docs_needed`: the documents still needed (usually [document categories](create-customer.md#document-categories)) |
| `note`         | string | Only with `more_docs_needed` or `kyb_rejected`, when the reviewer wrote one: what is missing, or the rejection reason |

| `status` | Meaning |
| -------- | ------- |
| `kyb_pending` | Under review |
| `approved` | Approved — orders can be created for this payer |
| `more_docs_needed` | Documents are needed: resubmit with [Create a customer](create-customer.md#resubmission) |
| `kyb_rejected` | Rejected; `note` may give the reason |

```json
{
  "id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
  "status": "more_docs_needed",
  "missing_docs": ["bank_statement"],
  "note": "bank statement older than 90 days"
}
```

An approved payer can return to `more_docs_needed` when its periodic re-review
is due; it then needs a full, current document set resubmitted, and no new
orders are accepted until it is approved again.

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `404` | `not_found` | Unknown id, or a payer created with another key |
| `500` | `get_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
