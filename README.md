# Redirect page

A dependency-free redirect page for GitHub Pages. Four layers, so it works
whether or not JavaScript runs.

| Layer | Mechanism | Runs when |
|---|---|---|
| 1 | `<meta http-equiv="refresh">` | Always — no JS required |
| 2 | `location.replace()` | JS enabled, and skips when the placeholder is unset |
| 3 | `<link rel="canonical">` | Crawlers / SEO only |
| 4 | Visible "Continue" link | JS fully blocked |

## Configure

There are **four** things to change in `index.html`:

1. `window.DESTINATION` — drives the JS layer (currently `""`, i.e. disabled)
2. `<link rel="canonical">` — crawler/SEO hint
3. `<meta http-equiv="refresh">` — the no-JS redirect
4. The visible **Continue** link in the body — the fallback when JS is blocked

Plus one optional: the rule in `_redirects` (Cloudflare/Netlify only).

Find all of them with:

```bash
grep -rn 'example.com/your-destination\|window.DESTINATION = ""' .
```

Static HTML cannot share one variable between a script and an attribute, so
items 2–4 must each be edited by hand. Missing item 4 is the common mistake:
the page redirects correctly for everyone until someone blocks JavaScript, at
which point the fallback link quietly sends them to the old destination.

`location.replace()` is used rather than `location.href` so the redirect does
not trap the visitor in Back-button history.

## Deploy

```bash
git init && git add -A && git commit -m "Add redirect page"
git branch -M main && git remote add origin git@github.com:<you>/<repo>.git
git push -u origin main
```

Then **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`**.

## Platform notes

- **GitHub Pages ignores `_redirects`.** The free plan has no static 301
  mechanism, so only the meta-refresh path applies. This file is there for
  Cloudflare Pages / Netlify if you move hosts.
- **`.nojekyll`** is required or Jekyll will skip `_redirects` and any
  underscore-prefixed file.
- First deploy takes 1–2 minutes; check `https://<you>.github.io/<repo>/`.

## Verify after deploy

```bash
set -o pipefail
curl -sIL https://<you>.github.io/<repo>/ | head -1     # expect HTTP/2 200
curl -sf https://<you>.github.io/<repo>/ | grep -o 'http-equiv="refresh"[^>]*'
```

`curl -f` plus `pipefail` keeps a 404 from being silently swallowed by the
pipe — without both, a missing page still prints `HTTP/2 404` and the command
appears to have worked.

## Caveats

- A redirect page provides **no** benefit for legitimate distribution and
  adds a hop that can be blocked or flagged independently of the target.
  Serving the file directly is simpler and more robust.
- If the target is a binary download, some corporate and school networks
  block redirects to executable content outright, so the direct link will
  reach more people than the redirect will.
