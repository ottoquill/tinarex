# Deployment — Cloudflare Workers Builds (static assets) via the GitHub app

This site deploys **automatically** on a git push. Nobody (and no agent)
deploys from a laptop. The flow:

```
git push  →  GitHub  →  Cloudflare GitHub app  →  Workers Builds runs:
                                                     npx wrangler deploy
                                                       └─ build.command:
                                                          git submodule update …
                                                          hugo --gc --minify
                                                       └─ upload ./public
                                                   →  live
```

Cloudflare created this as a **Workers** project (not a classic Pages
project). It deploys `public/` as a **static-assets-only Worker**. Everything
runs **inside Cloudflare's build environment**, triggered by the push — never
from a developer machine.

## The build is repo-driven (important)

[`wrangler.jsonc`](wrangler.jsonc) is committed and makes `npx wrangler
deploy` self-contained — it builds the site itself via `build.command`:

```jsonc
{
  "name": "tinarex",
  "compatibility_date": "2026-05-15",
  "build": {
    "command": "git submodule update --init --recursive && hugo --gc --minify"
  },
  "assets": { "directory": "public", "not_found_handling": "404-page" }
}
```

So you do **not** need to set a dashboard "Build command". Two earlier builds
failed because of that field:

| Symptom | Cause | Fixed by |
|---------|-------|----------|
| `npx hugo` → `npm error could not determine executable to run` | No `wrangler.jsonc`; wrangler auto-detected `npx hugo` (Hugo isn't an npm package) | Committing `wrangler.jsonc` |
| `assets.directory … does not exist: …/public` | `wrangler.jsonc` present, but nothing built `public/` (no build step ran) | `build.command` in `wrangler.jsonc` (builds before deploy) |

`build.command` runs *before* the assets are resolved, and `git submodule
update` guarantees the `hugo-book` theme is present even if Workers Builds
didn't fetch submodules.

## Dashboard settings (Cloudflare → Workers & Pages → `tinarex`)

Settings → **Builds**:

| Setting          | Value                 | Notes |
|------------------|-----------------------|-------|
| Git repository   | `ottoquill/tinarex`   | via the Cloudflare GitHub app |
| Production branch | `main` (or `setup` while iterating) | the branch whose builds go live |
| **Build command** | *(leave EMPTY)*      | `wrangler.jsonc`'s `build.command` does the build |
| Deploy command   | `npx wrangler deploy` | the default — leave as-is |

> If a dashboard Build command is set, it's harmless (Hugo just runs twice),
> but leaving it empty keeps the build defined in one place: `wrangler.jsonc`.

Environment variables (Settings → Variables, Production **and** Preview):

| Variable       | Value     | Why |
|----------------|-----------|-----|
| `HUGO_VERSION` | `0.159.0` | Pin Hugo ≥ 0.158.0 (theme minimum). Cloudflare auto-detects `hugo@extended_0.159.0`; pinning prevents a build-image change from moving it. |

After this, **Retry deployment** or push a commit.

## How deploys happen from now on

- **Push the production branch** → builds and updates the **live** site.
- **Push another branch / open a PR** → a **preview** deployment at a unique
  URL (if preview builds are enabled).
- No manual action, no local CLI, no secrets in the repo.

## After the first successful deploy

- Update `baseURL` in [`hugo.toml`](hugo.toml) to the real URL and push.
- **Custom domain** (optional): project → **Domains & Routes**.

## Verify locally before pushing (does NOT deploy)

```bash
git submodule update --init --recursive && hugo --gc --minify
```

This is exactly what `wrangler.jsonc`'s `build.command` runs in Cloudflare. It
only produces the local `public/` folder (gitignored); it never runs
`wrangler` and never uploads anything.

## Troubleshooting

- **`assets.directory … does not exist`** — `build.command` didn't run or
  didn't produce `public/`. Confirm `wrangler.jsonc` (with its `build` block)
  is committed on the branch being built; check the build log for the Hugo
  output.
- **`npx hugo` / `could not determine executable to run`** — `wrangler.jsonc`
  is missing on that branch, so wrangler auto-detected the wrong command.
- **Theme missing / empty `themes/hugo-book`** — the `git submodule update`
  in `build.command` covers this; if it still fails, check build-log network
  access to `github.com`.
- **Wrong Hugo version / unknown config keys** — set/raise `HUGO_VERSION` to
  match the theme minimum (≥ 0.158.0).
- **Old content after a push** — check the project's **Builds** /
  **Deployments** tab; the Cloudflare GitHub app may need re-authorization.
