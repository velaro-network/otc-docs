---
label: Create a payee
icon: person-add
order: 46
---

# Create a payee

:::code-tab POST
```
POST /api/v1/payees
```
:::

Registers a **payee** — the party abroad that a payer pays — with the bank
account it receives USD in. A payee belongs to one payer and is reused for
all of that payer's orders: create it once, not once per order.

Depending on how Velaro pays out for the payer, the bank account is either
registered for payouts up front — automatically, once the payer is approved —
or only when the order is paid out. In the first case an order can only be
created once `registration_status` is `registered`; in the second,
`registration_status` stays `pending` and does not block orders. You cannot
tell the two apart from this response, so do not wait on the field: once the
payer is approved, create the order, and if it answers `422`
`payee_not_registered`, try again later.

This call is **not idempotent**: each successful call creates a new payee.

## Request body

| Field                 | Type   | Required | Description |
| --------------------- | ------ | -------- | ----------- |
| `payer_id`            | string | yes | A payer created with your key |
| `relationship`        | string | yes | The payee's relationship to the payer — one of the values below |
| `bank_name`           | string | yes | Bank name, not blank |
| `account_holder_name` | string | yes | 1–35 characters (after trimming spaces). Letters, digits, spaces and ``!"#$%&'()+,-./:;<=>?@\_`~`` only — no non-ASCII script |
| `account_number`      | string | yes | Bank account number. All whitespace is removed |
| `bic`                 | string | yes | SWIFT/BIC code, not empty |
| `address_line1`       | string | yes | Account holder's address, 4–35 characters |
| `address_line2`       | string | no  | At most 35 characters |
| `city`                | string | yes | 1–35 characters |
| `country`             | string | yes | ISO 3166-1 **alpha-3** code, upper case (e.g. `USA`, `HKG`) |

`relationship` values: `self`, `holding_company`, `subsidiary_company`,
`branch_office`, `business_partner`, `supplier`, `customer`, `creditor`,
`debtor`, `franchisee`, `unknown`.

Use `self` only when the account belongs to the payer's own legal entity:
orders to any other payee require a transaction document
([Create an order](create-order.md)).

```json
{
  "payer_id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
  "relationship": "supplier",
  "bank_name": "Example Bank",
  "account_holder_name": "Acme Trading Ltd",
  "account_number": "123456789012",
  "bic": "EXAMUS33",
  "address_line1": "100 Main Street",
  "city": "New York",
  "country": "USA"
}
```

## Example

```bash
curl -X POST https://staging.payment.velaro.network/api/v1/payees \
  -H "X-API-Key: vcb_your_key" \
  -H "Content-Type: application/json" \
  -d '{"payer_id":"payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b","relationship":"supplier","bank_name":"Example Bank","account_holder_name":"Acme Trading Ltd","account_number":"123456789012","bic":"EXAMUS33","address_line1":"100 Main Street","city":"New York","country":"USA"}'
```

## Response `201`

The full account number is never returned.

| Field           | Type   | Description |
| --------------- | ------ | ----------- |
| `id`            | string | Payee id (`payee_` + 32 hex characters). Use it as `payee_id` when creating orders |
| `payer_id`      | string | The payer |
| `relationship`  | string | Echoed relationship |
| `account_last4` | string | Last 4 characters of the account number |
| `registration_status` | string | Registration of the bank account for payouts: `pending` (not registered yet — normal until the payer is approved, and permanent when the account is registered at payout time), `registering` (in progress), `registered` (ready for orders), `rejected` (the account details were refused — see `registration_error`), `needs_review` (Velaro is checking it) |
| `registration_error`  | string | Present only when a registration attempt failed: a short description of the problem |

```json
{
  "id": "payee_5d2c7a1b9e8f4a3c2b1d0e9f8a7b6c5d",
  "payer_id": "payer_8b1e4f2a9c3d4e5f6a7b8c9d0e1f2a3b",
  "relationship": "supplier",
  "account_last4": "9012",
  "registration_status": "pending"
}
```

A `rejected` account is not retried; contact [support](../support.md) to
correct the bank details.

## Errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `400` | `bad_request` | Body is not valid JSON or a field has the wrong JSON type |
| `400` | `invalid_request` | A field fails its rule. Carries `field` (the field name) and `reason`: `required`, `too_short`, `too_long`, `invalid_characters`, `invalid_format` or `invalid_value` |
| `404` | `not_found` | `payer_id` unknown or created with another key |
| `409` | `conflict` | Concurrent update — retry |
| `413` | `payload_too_large` | Body over 64,000 bytes |
| `500` | `create_payee_failed` | Unexpected failure |

Plus the codes common to every endpoint — see [Errors](../errors.md).

```json
{
  "status": 400,
  "title": "Bad Request",
  "code": "invalid_request",
  "detail": "payee: invalid input: country must be an ISO 3166-1 alpha-3 code (e.g. HKG)",
  "field": "country",
  "reason": "invalid_format"
}
```
