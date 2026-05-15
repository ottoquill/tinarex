# CLAUDE.md — Working rules for the Tina Tiny T. Rex site

This file is instructions for Claude (and any AI agent) working in this repo.
Read it before making changes.

## What this project is

A children's storybook website about **Tina Tiny T. Rex**, the smallest T. Rex
in Green Valley. Built with **Hugo** + the **hugo-book** theme, deployed by
**CloudFlare Pages** via its GitHub app integration.

Audience: **early readers, ages 6–9** (also read aloud by a parent).

## ⛔ Deployment guardrail (most important rule)

**Never deploy from this machine. Never run a deploy.**

This site is a Cloudflare **Workers static-assets** project (Workers Builds),
configured by [`wrangler.jsonc`](wrangler.jsonc). On a git push, Cloudflare
runs `npx wrangler deploy`, which itself runs the build via the
`build.command` in `wrangler.jsonc` (`git submodule update … && hugo --gc
--minify`) and then uploads `public/`. All **inside Cloudflare's build
environment** — never on a developer machine. No dashboard "Build command"
is needed.

- Do **not** run `wrangler deploy`, `wrangler dev`, `hugo deploy`, or any
  command that uploads the site. `wrangler` runs in Cloudflare's build, here.
- Do **not** add GitHub Actions / CI that builds or deploys.
- The **only** deploy path: push to GitHub → Cloudflare's GitHub app sees the
  push → Workers Builds runs the build + `wrangler deploy`. See
  `DEPLOYMENT.md`.
- Pushing the **`main`** branch (the production branch) publishes the **live**
  site. Treat `main` with care. Do feature work on a branch; let the user
  merge.
- Building locally with `hugo` is for **preview/verification only** — that
  output (`public/`) is gitignored and must never be committed.
- [`wrangler.jsonc`](wrangler.jsonc) **must** stay committed. If it's missing,
  wrangler auto-detects `npx hugo` as the build command and the deploy fails
  (Hugo is a binary, not an npm package). Don't delete or .gitignore it.

If asked to "deploy", explain that deployment happens automatically on push and
that you cannot and should not trigger it directly.

## Writing style guide (ages 6–9)

Tina's stories must stay readable by a 6–9 year old:

- **Short sentences.** One idea per sentence. Short paragraphs (1–3 sentences).
- **Simple, concrete words.** Prefer "tiny" over "minuscule", "scared" over
  "apprehensive". If a word is rare, the meaning must be clear from context.
- **Warm and gentle.** Tension is okay (someone stuck, lost, sad) but it
  resolves kindly. No real peril, violence, or scary endings.
- **Rhythm and repetition** help early readers: e.g. "Step. Step. Step."
- **Show feelings simply:** "Tina felt small." not "Tina was overcome with
  inadequacy."
- **Sound words** are encouraged: *crack, tap, squeak, bzzz.*
- Every story should leave the reader with a **kind, hopeful idea**.

### Character bible (keep consistent)

- **Tina** — the smallest T. Rex in Green Valley. Brave, kind, quietly worried
  about being small; learns her size is a strength. Roar = a "squeak".
- **Pip** — a cheerful dragonfly, Tina's first friend. Optimistic; his wings
  give a soft glow in the dark. Catchphrase idea: small is "just me".
- **Mama Rex** — warm, patient. "You are exactly the right size to be *you.*"
- **Rosie** — a little Triceratops; curious, gets into small scrapes.
- **The big ones** — Tina's siblings/valley dinosaurs: loud, strong, kind,
  unable to do small things.
- **Setting** — Green Valley: ferns, rivers, cliffs, warm mornings.
- **Theme** — being small/different is a strength; friendship; bravery.

## Repository structure

```
hugo.toml                 Site + hugo-book config (single language)
content/_index.md         Landing/welcome page ("Start the story")
content/docs/_index.md    Story index (BookSection = "docs"); table of chapters
content/docs/chapter-NN-*.md   One file per chapter
themes/hugo-book/         Theme — git submodule, do NOT edit in place
static/                   Images and other static assets (create as needed)
DEPLOYMENT.md             CloudFlare Pages setup (one-time, dashboard)
```

## How to add or edit a chapter

1. Create `content/docs/chapter-NN-short-slug.md` (zero-padded NN).
2. Front matter:
   ```yaml
   ---
   title: "Chapter N: Human Title"
   weight: N        # controls sidebar + reading order
   ---
   ```
3. Start the body with an `# Chapter N: ...` H1 matching the title.
4. End the chapter with a "Next" button to the following chapter:
   ```
   {{</* button href="/docs/chapter-NN-next-slug" */>}}Next: Title →{{</* /button */>}}
   ```
   The final chapter instead ends with two buttons: `href="/"` ("← Back to
   the beginning") and `href="/docs/chapter-01-a-very-tiny-egg"` ("Read it
   again").
5. Update the chapter list in `content/docs/_index.md` (use the
   `relref` *shortcode* there — see linking rules below).
6. Follow the writing style guide above. Keep chapters ~250–400 words.

### Linking rules (important — easy to get wrong)

This theme's `button` shortcode and its `relref` shortcode are **not** the
same and do not take the same argument:

- **`button` shortcode → use `href` only.** It runs the value through the
  theme's `portable-link` partial. Pass an **absolute content path**:
  `href="/docs/chapter-02-too-small-to-play"` (no `.md`, no trailing slash) or
  `href="/"` for home. **`relref="..."` on a `button` is silently ignored and
  the link falls back to `/`.** (The theme's exampleSite docs show
  `button relref=` — that documentation is stale for this version. Trust the
  rendered HTML, not the example.)
- **Inline Markdown links → use the `relref` shortcode:**
  `[Chapter 1]({{</* relref "chapter-01-a-very-tiny-egg" */>}})` (filename,
  no extension). This is the standard Hugo shortcode and works correctly; it
  errors loudly if the target is missing.

After any link change, **rebuild and grep the HTML** to confirm buttons point
at real chapter URLs, not `/`:
`hugo --gc --minify && grep -o 'book-btn[^<]*' public/docs/*/index.html`

## Local preview & verification

- Preview with drafts: `hugo server -D` → open http://localhost:1313
- **Always verify a production build before pushing:**
  `hugo --gc --minify` — it must finish with **0 errors** and no broken
  `relref`/`ref` warnings.
- Never commit `public/` or `resources/_gen/` (gitignored).

## Theme (hugo-book) is a git submodule

- Added at `themes/hugo-book`. Cloudflare Workers Builds clones submodules
  automatically, so no extra deploy config is needed for the theme.
- Don't hand-edit files under `themes/hugo-book/`. To customize, override the
  layout/partial by copying it into the project root `layouts/` directory.
- After `git clone`, run `git submodule update --init --recursive`.
- Hugo must be **≥ 0.158.0** (theme requirement). Keep `HUGO_VERSION` in
  CloudFlare in sync with the local Hugo version.

## Git workflow

- Branch for changes; never push straight to `main` unless the user asks.
- Commit/push only when the user explicitly asks (pushing `main` = live deploy).
- Keep `themes/hugo-book` submodule pointer changes in their own commit when
  bumping the theme.
