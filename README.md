# Tina Tiny T. Rex 🦖

A children's storybook website about **Tina**, the smallest T. Rex in Green
Valley — for early readers, ages 6–9.

Built with [Hugo](https://gohugo.io/) and the
[hugo-book](https://github.com/alex-shpak/hugo-book) theme. Deployed by
**Cloudflare Workers Builds** automatically on every push (see
[`DEPLOYMENT.md`](DEPLOYMENT.md)).

## Get started

```bash
git clone git@github.com:ottoquill/tinarex.git
cd tinarex
git submodule update --init --recursive   # fetches the hugo-book theme
hugo server -D                            # http://localhost:1313
```

Requires **Hugo extended ≥ 0.158.0**.

## Where things live

- `content/_index.md` — welcome page
- `content/docs/` — the story, one file per chapter
- `hugo.toml` — site + theme configuration
- `themes/hugo-book/` — theme (git submodule, don't edit in place)

## Adding a chapter

See [`CLAUDE.md`](CLAUDE.md) — it has the chapter template, the writing style
guide for this age group, and the character bible. Read it before contributing
(human or AI).

## Deployment

You don't deploy by hand. Cloudflare Workers Builds builds and deploys on
push. Push to a branch for a **preview**, or to the production branch for the
**live** site. Details in [`DEPLOYMENT.md`](DEPLOYMENT.md).
