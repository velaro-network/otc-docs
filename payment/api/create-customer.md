---
label: Create a customer (KYB)
icon: organization
order: 48
---

# Create a customer (KYB)

:::code-tab POST
```
POST /api/v1/customers
```
:::

Creates a **payer** — the Brazilian business that pays — and submits its KYB
documents for review. The same endpoint **resubmits** documents for an
existing payer (send its `id`) after a `more_docs_needed` result.

Follow the review with [Customer status](customer-status.md) or the
`payer.review_result` [webhook](../webhooks.md). Orders can only be created
for an approved payer.

## Request body

| Field        | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| `id`         | string | no  | An existing payer's id. Present: resubmit documents for that payer. Absent: create a new payer |
| `legal_name` | string | yes | Company legal name. Required on resubmission too, but then ignored (it cannot change) |
| `cnpj`       | string | yes | Company CNPJ, 14 characters once `.`, `/`, `-` and spaces are removed (digits, or letters for the alphanumeric CNPJ), with valid check digits. Required on resubmission too, but then ignored |
| `profile`    | object | no  | The remaining KYB details (registered address, shareholders, directors, declarations, …) as a JSON object, up to 64 KiB. On resubmission, replaces the stored profile when sent |
| `documents`  | array  | yes | At least 1, at most 40 documents |
| `documents[].category`       | string | yes | One of the categories below |
| `documents[].filename`       | string | yes | File name, 1–255 characters |
| `documents[].content_type`   | string | no  | Ignored: the type is detected from the file's bytes |
| `documents[].content_base64` | string | yes | File content, standard base64. PDF, JPEG, PNG, DOC or DOCX, at most 50,000,000 bytes (50 MB) decoded. DOC and DOCX are re-rendered to PDF |

All documents of one submission together: at most 250,000,000 bytes (250 MB)
decoded. No text field may contain a NUL (`\u0000`) character. See
[Limits](../limits.md) for every size rule.

!!!warning Availability
The 50 MB per-document limit and the DOC and DOCX formats take effect only
where document conversion is enabled. Until then, a document is at most
10 MB decoded and must be PDF, JPEG or PNG.
!!!

### Document categories

`contact_id_document`, `contact_selfie`, `regulatory_license`,
`shareholder_id_document`, `shareholder_selfie`,
`shareholder_proof_of_address`, `director_id_document`, `director_selfie`,
`director_proof_of_address`, `articles_of_incorporation`,
`financial_statements`, `shareholder_declaration`, `ownership_chart`,
`proof_of_business_address`, `bank_statement`, `power_of_attorney`,
`aml_policy`, `kyc_policy`, `trade_contract`, `trade_invoice`,
`trade_supporting_document`, `w8_ben_e_document`, `proof_of_source_of_funds`,
`board_resolution`, `other`.

Categories are case-sensitive.

### Resubmission

Accepted only while the payer's status is `more_docs_needed`. The new
documents are added to those on file (nothing is replaced) and the reviews
that asked for them start again (normally back to `kyb_pending`). A payer may hold at most 1,000 documents across
all submissions.

```json
{
  "legal_name": "Exemplo Importadora Ltda",
  "cnpj": "11.222.333/0001-81",
  "profile": { "registered_address": { "city": "São Paulo", "country": "BRA" } },
  "documents": [
    {
      "category": "articles_of_incorporation",
      "filename": "contrato-social.pdf",
      "content_type": "application/pdf",
      "content_base64": "JVBERi0xLjQK..."
    }
  ]
}
```

## Example

```bash
curl -X POST https://staging.payment.velaro.network/api/v1/customers \
  -H "X-API-Key: vcb_your_key" \
  -H "Content-Type: application/json" \
  -d @customer.json
```

## Response `201`

Returned for a new payer and for a resubmission.

| Field          | Type   | Description |
| -------------- | ------ | ----------- |
| `id`           | string | Payer id (`payer_` + 32 hex characters). Use it as `payer_id` elsewhere |
| `status`       | string | KYB status — see [Customer status](customer-status.md). `kyb_pending` after a submission |
| `missing_docs` | array  | Only with `more_docs_needed`: the documents still needed |
| `note`         | string | Only with `more_docs_needed` or `kyb_rejected`, when the reviewer wrote one |

```json
{
  "id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
  "status": "kyb_pending"
}
```

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `bad_request` | Body is not JSON, `Content-Type` is not `application/json`, or a field has the wrong JSON type |
| `400` | `invalid_request` | A field fails its rule: `legal_name`/`cnpj` missing, invalid CNPJ, no documents, more than 40 documents, unknown category, missing or too-long filename, a document over 50 MB or not PDF/JPEG/PNG/DOC/DOCX, invalid base64, submission over 250 MB, `profile` not an object or over 64 KiB, a field over its size limit, a NUL character, or more than 1,000 documents for the payer |
| `404` | `not_found` | Resubmission: `id` unknown or created with another key |
| `409` | `conflict` | A payer with this CNPJ already exists; or a resubmission that is not accepted (payer not in `more_docs_needed`, or rejected); or a concurrent update — retry |
| `413` | `payload_too_large` | Body over 340,000,000 bytes |
| `503` | `service_busy` | Too many uploads in progress — retry after the `Retry-After` interval (see [Limits](../limits.md#kyb-uploads-in-progress)) |
| `500` | `create_customer_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).
