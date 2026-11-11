---
label: Authentication
icon: key
order: 80
---

# Authentication

Every request requires a Velaro API key.

## Getting a key

API keys are issued by Velaro. **Contact your account manager** (or
[support](support.md)) to have a key provisioned for your organization. Keys
look like `vcb_…` and are shown to you **once** — store them securely.

## Using your key

Send the key in the `X-API-Key` header:

```bash
curl "https://staging-payment.velaro.network/api/v1/cross-border/partner/rates?payout_amount=1000&payout_currency=USD" \
  -H "X-API-Key: vcb_your_key"
```

A Bearer token is also accepted:

```bash
curl "https://staging-payment.velaro.network/api/v1/cross-border/partner/rates?payout_amount=1000&payout_currency=USD" \
  -H "Authorization: Bearer vcb_your_key"
```

Rules:

- If `X-API-Key` is present (and not blank) it is the key used; `Authorization`
  is then ignored.
- `Authorization: Bearer` is only read when the token starts with `vcb_`.
- Leading and trailing spaces around the key are ignored.

## Your data is scoped to your key

Payers, payees, quotes and orders belong to the key that created them. Using
another key's id answers `404 not_found`, exactly as an id that does not
exist.

## Authentication errors

| Status | `code` | When |
| ------ | ------ | ---- |
| `401` | `missing_api_key` | No key sent |
| `401` | `invalid_api_key` | Key unknown or revoked |
| `403` | `scope_forbidden` | Valid key, but not issued for this API |

See [Errors](errors.md) for the response shape.

## Keeping keys safe

- Treat the key like a password — never commit it or expose it in client-side code.
- If a key is compromised, contact Velaro to **revoke** it; revoked keys return `401`.
- You may hold multiple keys (e.g. per environment); revoke individually. Data
  created with one key is not visible with another.
