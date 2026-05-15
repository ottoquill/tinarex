# Deployment — CloudFlare Pages via the GitHub app

This site deploys **automatically**. Nobody (and no agent) deploys from a
laptop. The flow is:

```
git push  →  GitHub  →  CloudFlare Pages GitHub app  →  build  →  live
```

There is **no** GitHub Actions workflow and **no** `wrangler` step on purpose.
CloudFlare Pages watches the repo through its GitHub app and does the build
itself.

## One-time setup (CloudFlare dashboard)

Do this once, in the CloudFlare dashboard. It is a manual, human step.

1. **Connect GitHub**
   - CloudFlare dashboard → **Workers & Pages** → **Create** → **Pages** →
     **Connect to Git**.
   - Authorize the **CloudFlare Pages GitHub app** and grant it access to the
     `ottoquill/tinarex` repository (repo-scoped access is fine).
   - Select `ottoquill/tinarex`.

2. **Build settings**

   | Setting                   | Value             |
   |---------------------------|-------------------|
   | Production branch         | `main`            |
   | Framework preset          | `Hugo`            |
   | Build command             | `hugo --gc --minify` |
   | Build output directory    | `public`          |
   | Root directory            | `/` (default)     |

3. **Environment variables** (Settings → Environment variables → Production
   *and* Preview):

   | Variable        | Value     | Why |
   |-----------------|-----------|-----|
   | `HUGO_VERSION`  | `0.159.0` | Pin Hugo to a known-good version. Keep this in sync with local Hugo and the theme's minimum (≥ 0.158.0). |

   CloudFlare clones **git submodules automatically**, so the `hugo-book`
   theme is fetched with no extra configuration.

4. **Save and deploy.** CloudFlare runs the first build. When it finishes, the
   site is live at `https://<project>.pages.dev`.

## After the first deploy

- Update `baseURL` in [`hugo.toml`](hugo.toml) to the real URL (the
  `*.pages.dev` address, or a custom domain if you add one) and push the change.
- **Custom domain** (optional): Pages project → **Custom domains** → add the
  domain and follow the DNS instructions.

## How deploys happen from now on

- **Push to `main`** → CloudFlare builds and updates the **production** site.
- **Push to any other branch** (or open a PR) → CloudFlare builds a **preview
  deployment** at a unique URL, so changes can be reviewed before they go live.
- No manual action, no CLI, no secrets stored in the repo.

## Verify a build locally before pushing (does NOT deploy)

```bash
hugo --gc --minify   # must finish with 0 errors / no broken refs
```

This only produces the local `public/` folder (gitignored). It never uploads
anything — it just proves CloudFlare's build will succeed.

## Troubleshooting

- **Build fails: theme missing / empty `themes/hugo-book`** — the submodule
  wasn't fetched. CloudFlare does this automatically; if a build is from a
  manual upload instead of the Git integration, switch it back to Git.
- **Build fails: Hugo version / unknown config** — bump `HUGO_VERSION` in
  CloudFlare to match the local Hugo version and the theme minimum.
- **Old content after a push** — check the Pages **Deployments** tab; the
  GitHub app may need re-authorization if it lost repo access.
