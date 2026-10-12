# otc-docs

Developer documentation for the **Velaro OTC Quote API**, served at
`https://docs.velaro.network`.

Built with [Retype](https://retype.com). Content is plain Markdown; the API
reference under `api/` is hand-written (one page per endpoint).

## Develop

```bash
npm install
npm run dev      # retype watch → http://localhost:5000
```

## Build

```bash
npm run build    # static site in ./.retype
```

> If `retype` isn't on PATH after install, run it via the package:
> `node node_modules/retypeapp/retype.js build`.

## Structure

- `retype.yml` — site config (branding, nav, section order).
- `index.md`, `overview.md`, `authentication.md`, `rate-limits.md`, `errors.md`, `support.md` — prose.
- `api/` — API reference, one page per endpoint.
- `static/` — logo and favicon.

## Deploy (Vercel)

No `vercel.json`; configure in the Vercel dashboard:

- **Build Command:** `retype build`
- **Output Directory:** `.retype`
- **Install Command:** `npm install`

Push to the repo → Vercel builds and deploys. Add the custom domain under
**Settings → Domains** and point DNS at the CNAME Vercel provides. Test on the
`*.vercel.app` URL first, then bind `docs.velaro.network`.

## Support email

The address in the docs (`support@velaro.network`) is a placeholder — replace it
in `support.md` and the API pages once confirmed.
