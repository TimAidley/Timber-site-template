# Timber starter — fork and go

A complete, working Timber **site** you can stand up with **no local setup**: fork this
repo, add a few secrets, run one Action, and you get a live website on GitHub Pages
**plus** an in-browser editor at `/<repo>/admin/`. (SPEC §3: a site is a thin host that
pins a Timber version + its config — not a copy of the app source.)

- **Your site** → `https://<you>.github.io/<repo>/`
- **The editor** → `https://<you>.github.io/<repo>/admin/`

> Prefer a local workflow, or want to understand the moving parts? See **INSTALL.md** in
> the Timber repo (includes a paste-a-PAT quick-start with zero cloud setup).

---

## Setup (all in the browser)

### 1. Use this template
Click **“Use this template” → Create a new repository** (or Fork). Name it whatever you
like; your site will live at `https://<you>.github.io/<that-name>/`.

### 2. Enable Pages
**Settings → Pages → Source: “GitHub Actions.”** (The first deploy also tries to enable
this automatically, but setting it now avoids a first-run hiccup.)

### 3. Register + install a GitHub App
This lets the editor sign you in with least-privilege, per-repo access. **GitHub →
Settings → Developer settings → GitHub Apps → New GitHub App:**
- **Callback URL:** `https://<you>.github.io/<repo>/admin/` (exactly — trailing slash).
- **Expire user authorization tokens:** checked (recommended).
- **Webhook → Active:** unchecked (if the form demands a URL, put `https://example.com`).
- **Where can this be installed?** Only on this account.
- **Repository permissions:** **Contents: Read & write**, **Actions: Read & write**,
  **Metadata: Read** (auto). Nothing else.
- Create it, copy the **Client ID**, generate a **Client secret**, then **Install App**
  on **this repo** (Only select repositories). The install is what grants repo access.

> Prefer a classic **OAuth App**? It works too — register one with the same callback URL
> and use the same secret names below; it just grants account-wide `repo` scope instead of
> per-repo access. Full step-by-step + troubleshooting: **INSTALL.md** in the Timber repo.

### 4. Add secrets + variables
In **your** repo → **Settings → Secrets and variables → Actions** (the `GH_` prefix
matters — GitHub forbids names starting with `GITHUB_`):

| Kind | Name | Value |
|---|---|---|
| Variable | `GH_OAUTH_CLIENT_ID` | the App's **client id** (public) |
| Secret | `GH_OAUTH_CLIENT_SECRET` | the App's **client secret** |
| Secret | `CLOUDFLARE_API_TOKEN` | a Cloudflare token with **Workers Scripts: Edit** |
| Secret | `CLOUDFLARE_ACCOUNT_ID` | your Cloudflare **account id** |

(Cloudflare account id + API token come from the Cloudflare dashboard → Workers. The
free plan is enough. Make sure your account has a **workers.dev subdomain** enabled.)

### 5. Set your base URL
Edit `content/settings/index.md` → `baseUrl: https://<you>.github.io/<repo>` (no trailing
slash). This makes links, canonical URLs, and the sitemap correct under your repo's
subpath. Commit it.

### 6. Run “Setup OAuth broker”
**Actions → “Setup OAuth broker” → Run workflow.** It deploys the Cloudflare broker,
records its URL in `.timber-broker-url`, and kicks off a deploy. When the **Build &
deploy site** run goes green, your site is live and the editor works at
`https://<you>.github.io/<repo>/admin/`.

That's it — open the editor, **Sign in with GitHub**, and start writing. Edits autosave
to a `‹you›_wip` branch; **Publish** merges to `main`, which auto-deploys.

---

## What's here

- **`templates/default.liquid`** + **`assets/theme.css`** — the default theme (SPEC §13):
  SEO `<head>`, top nav, page body, footer. Customize by editing these; add
  `templates/<type>.liquid` to override per content type. In-page links are prefixed with
  `{{ site.basePath }}` so the theme works under a project-Pages subpath.
- **`config/schemas/*.yml`** — content types (`pages`, plus a `settings` singleton with
  `page: false` holding site-wide identity + naming the `homepage`).
- **`config/navigation.yml`** — the ordered top navigation.
- **`content/**`** — your content bundles. `content/settings/index.md` is global settings;
  the object it names as `homepage` renders at the site root.
- **`.github/workflows/`** — `deploy.yml` (build the site + co-host the editor → Pages) and
  `setup-broker.yml` (one-time Cloudflare broker deploy). Pin `TIMBER_REF` in both to a
  Timber release tag once you cut one, so preview ≡ production.

## Build locally (optional)

```
timber build . _site   # renders the whole site into ./_site
```
