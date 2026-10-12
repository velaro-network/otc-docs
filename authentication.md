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
look like `vk_live_…` and are shown to you **once** — store them securely.

## Using your key

Send the key as a Bearer token:

```bash
curl https://otc-api.velaro.network/v1/rates \
  -H "Authorization: Bearer vk_live_your_key"
```

The `X-API-Key` header is also accepted:

```bash
curl https://otc-api.velaro.network/v1/rates \
  -H "X-API-Key: vk_live_your_key"
```

## Keeping keys safe

- Treat the key like a password — never commit it or expose it in client-side code.
- If a key is compromised, contact Velaro to **revoke** it; revoked keys return `403`.
- You may hold multiple keys (e.g. per environment); revoke individually.
