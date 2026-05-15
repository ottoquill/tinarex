# Deployment — Cloudflare Workers Builds (static assets) via the GitHub app

This site deploys **automatically**. Nobody (and no agent) deploys from a
laptop. The flow is:

```
git push  →  GitHub  →  Cloudflare GitHub app  →  Workers Builds:
                                                     hugo --gc --minify
                                                     npx wrangler deploy
                                                   →  live
```

Cloudflare created this as a **Workers** project (not a classic Pages
project). It builds the Hugo site and deploys `public/` as a
**static-assets-only Worker**. The deploy mechanism (`wrangler`) runs **inside
Cloudflare's build environment**, triggered by the git push — never from a
developer machine.

## The repo half (already done)

[`wrangler.jsonc`](wrangler.jsonc) is committed and tells `wrangler deploy`
exactly what to ship:

```jsonc
{
  "name": "tinarex",
  "compatibility_date": "2026-05-15",
  "assets": { "directory": "public", "not_found_handling": "404-page" }
}
```

**Why this file matters:** without it, `npx wrangler deploy` runs in
auto-detection mode, guesses the build command is `npx hugo`, and fails with
`npm error could not determine executable to run` (Hugo is a native binary,
not an npm package). Keep `wrangler.jsonc` committed.

## The dashboard half (one-time, do this in Cloudflare)

Cloudflare dashboard → **Workers & Pages** → the **`tinarex`** project →
**Settings** → **Builds**:

| Setting          | Value                | Notes |
|------------------|----------------------|-------|
| Git repository   | `ottoquill/tinarex`  | via the Cloudflare GitHub app |
| Production branch | `main` (or `setup` for now) | the branch whose builds go live |
| Build command    | `hugo --gc --minify` | **change this** — it was auto-set to the broken `npx hugo` |
| Deploy command   | `npx wrangler deploy`| reads `wrangler.jsonc`, uploads `public/` |
| Build output     | *(leave blank)* | `wrangler.jsonc`'s `assets.directory` controls this |

Environment variables (Settings → Variables, for Production **and** Preview):

| Variable       | Value     | Why |
|----------------|-----------|-----|
| `HUGO_VERSION` | `0.159.0` | Pin Hugo. Must be ≥ 0.158.0 (theme minimum) and match local Hugo. Cloudflare detected `hugo@extended_0.159.0` automatically, but pin it so a build-image change can't move it. |

Submodules (the `hugo-book` theme) are cloned by Workers Builds automatically —
no extra configuration.

Save, then **Retry deployment** (or push a commit). A successful build ends
with a `…workers.dev` (or custom-domain) URL.

## What the *failed* build looked like (for reference)

```
Executing user deploy command: npx wrangler deploy
 - Build Command: npx hugo            ← wrong (auto-detected, no wrangler.jsonc)
[build] Running: npx hugo
[build] npm error could not determine executable to run   ← Hugo isn't npm
✘ Running custom build `npx hugo` failed.
```

Both halves above fix this: `wrangler.jsonc` stops the bad auto-detection, and
the dashboard build command becomes `hugo --gc --minify`.

## How deploys happen from now on

- **Push to the production branch** → Workers Builds builds and updates the
  **live** site.
- **Push to any other branch / open a PR** → a **preview** build at a unique
  URL, for review before it goes live (if preview builds are enabled for the
  project).
- No manual action, no local CLI, no secrets in the repo.

## After the first successful deploy

- Update `baseURL` in [`hugo.toml`](hugo.toml) to the real URL and push.
- **Custom domain** (optional): project → **Domains & Routes** → add the
  domain and follow the DNS instructions.

## Verify a build locally before pushing (does NOT deploy)

```bash
hugo --gc --minify   # must finish with 0 errors / no broken refs
```

This only produces the local `public/` folder (gitignored). It never runs
`wrangler` and never uploads anything — it just proves the build half will
succeed in Cloudflare.

## Troubleshooting

- **`npm error could not determine executable to run` / `npx hugo` fails** —
  the dashboard build command is still `npx hugo`. Change it to
  `hugo --gc --minify`. Confirm `wrangler.jsonc` is committed on the branch
  being built.
- **`wrangler deploy` can't find assets / deploys nothing** — `public/`
  wasn't built. The build command must run **before** the deploy command and
  must be `hugo --gc --minify` (not `npx hugo`).
- **Theme missing / empty `themes/hugo-book`** — submodule not fetched;
  ensure the project uses the Git integration (Workers Builds fetches
  submodules automatically).
- **Wrong Hugo version / unknown config keys** — set/raise `HUGO_VERSION` to
  match local Hugo and the theme minimum (≥ 0.158.0).
- **Old content after a push** — check the project's **Deployments** /
  **Builds** tab; the Cloudflare GitHub app may need re-authorization if it
  lost repo access.
