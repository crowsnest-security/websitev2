# CrowsNest — websitev2 (static site)

This repository hosts the **static HTML** public website for **CrowsNest Systems**, deployed to **GitHub Pages** with the custom apex domain **`crowsnestsecurity.com`**.

It is intended to **replace** the legacy Docusaurus site in [`crowsnest-security/crowsnest-security.github.io`](https://github.com/crowsnest-security/crowsnest-security.github.io). After cutover, that repository should be **archived** and its README updated to point here.

There is **no Docusaurus** (or React) in this project: only files under `public/` are published.

---

## Repository layout

| Path | Purpose |
|------|---------|
| [`public/`](public/) | **Entire website**: HTML, assets, and the [`public/CNAME`](public/CNAME) file for the custom domain. |
| [`package.json`](package.json) | Node scripts: local preview and manual deploy via [`gh-pages`](https://www.npmjs.com/package/gh-pages). |
| [`.github/workflows/deploy-gh-pages.yml`](.github/workflows/deploy-gh-pages.yml) | CI: on every push to `main`, publishes `public/` to the `gh-pages` branch. |
| [`.github/dependabot.yml`](.github/dependabot.yml) | Weekly Dependabot PRs for npm dependencies. |

---

## Prerequisites

- **Node.js 18+** (see `engines` in [`package.json`](package.json)).
- Git and npm.
- Push access to [`crowsnest-security/websitev2`](https://github.com/crowsnest-security/websitev2).

---

## Local development

Install dependencies (once):

```bash
npm install
```

Preview the static site (serves the `public/` folder on port 3000):

```bash
npm run serve
```

Open `http://localhost:3000` in your browser. Edit files under `public/` and refresh.

---

## How deployment works

### Primary path: GitHub Actions (recommended)

1. Merge changes into **`main`** (for example via pull request).
2. The workflow **[Deploy to gh-pages](.github/workflows/deploy-gh-pages.yml)** runs automatically.
3. It pushes the contents of **`public/`** to the **`gh-pages`** branch (using [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)).
4. GitHub Pages serves the site from that branch (see [Repository settings](#repository-settings-github-ui) below).

This matches the spirit of the old Docusaurus repo’s “merge then deploy” flow, without running a framework build.

### Optional: manual deploy from your machine

Use this if you need to publish without waiting for CI (for example troubleshooting):

```bash
npm run deploy
```

This runs `gh-pages -d public -b gh-pages`, which builds nothing; it only copies `public/` to the `gh-pages` branch.

If Git reports:

```text
Error: Please set the GIT_USER environment variable, or explicitly specify USE_SSH instead!
```

use SSH (same idea as the legacy site):

```bash
USE_SSH=true npm run deploy
```

Avoid running **manual deploy** immediately after a merge that already triggered a successful workflow unless you intend to overwrite the branch again with the same content.

---

## Repository settings (GitHub UI)

Configure these in **Settings** for `crowsnest-security/websitev2`.

### Pages

1. Go to **Settings → Pages**.
2. **Build and deployment → Source**: **Deploy from a branch** (not “GitHub Actions” as the publishing source, unless you later switch to artifact-based Pages).
3. **Branch**: **`gh-pages`**, folder **`/ (root)`**.
4. **Custom domain**: enter **`crowsnestsecurity.com`** (apex), then save.
5. After DNS validates, enable **Enforce HTTPS** when GitHub allows it (can take up to 24 hours).

GitHub may redirect between apex and `www` automatically when DNS for both is correct. This repo’s canonical hostname is the apex; [`public/CNAME`](public/CNAME) contains `crowsnestsecurity.com`.

### Actions and permissions

- **Settings → Actions → General**: ensure Actions are allowed for the repository (and for the organization, if policies apply).
- Workflows need permission to push to `gh-pages`; the included workflow sets `permissions: contents: write` on the deploy job.

### Organization settings

If Pages does not appear or builds fail org-wide, an org owner may need to allow GitHub Pages for public repositories (**Organization settings → Pages**).

---

## DNS (GoDaddy or any provider)

Configure DNS so traffic reaches **GitHub Pages**. Values below match GitHub’s current documentation:

**Official reference:** [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

### Apex `crowsnestsecurity.com` (required)

Create **four `A` records** on `@` (apex):

| Type | Name | Data |
|------|------|------|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

Optional **IPv6**: add **four `AAAA` records** on `@`:

| Type | Name | Data |
|------|------|------|
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

Remove any **old** apex `A` records that pointed to non-GitHub hosts (for example previous AWS IPs).

### `www` subdomain (recommended)

Create a **`CNAME`** for **`www`** pointing to the organization Pages hostname (no repository name in the target):

| Type | Name | Data |
|------|------|------|
| CNAME | `www` | `crowsnest-security.github.io` |

Do **not** point `www` at the apex only (for example `crowsnestsecurity.com.`); GitHub expects `www` → `ORGANIZATION.github.io` for HTTPS and redirects.

### Email (do not break)

Keep **MX**, **SPF** (`TXT`), **DKIM**, and **DMARC** records for Google Workspace (or other mail) **unchanged**.

### CAA

If you use **CAA** records, GitHub Pages certificates are issued via Let’s Encrypt. Allow issuance for `letsencrypt.org` (for example `0 issue "letsencrypt.org"`), or GitHub’s documentation may list additional CAs—see [Troubleshooting custom domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages) if certificate provisioning fails.

DNS can take up to **24–48 hours** to propagate. Use `dig` as described in GitHub’s docs to verify.

---

## Custom domain and `CNAME` file

- The file [`public/CNAME`](public/CNAME) contains a single line: `crowsnestsecurity.com`.
- Each deploy copies it to the root of the **`gh-pages`** branch so the custom domain association survives publishes.
- You should still set the **same** domain under **Settings → Pages → Custom domain** so GitHub can issue TLS and show the correct checks.

If the custom domain disappears from Settings after a bad deploy, re-save the domain in the Pages UI and confirm `public/CNAME` is present on `gh-pages`.

---

## Relationship to the old Docusaurus repository

| Old (`crowsnest-security.github.io`) | New (`websitev2`) |
|-------------------------------------|-------------------|
| Docusaurus build → `build/` → `gh-pages` | Static files in `public/` → `gh-pages` |
| `npm run deploy` (Docusaurus) | `npm run deploy` (`gh-pages` CLI) or push to `main` |
| Depended on dynamic Actions + Dependabot | Adds checked-in **Dependabot** + **deploy workflow** |

After traffic uses this site and DNS is stable, **archive** the old repository and add a short pointer in its README to this repo.

---

## Troubleshooting

| Issue | What to check |
|-------|----------------|
| Site shows “404” on GitHub.io URL | **Pages** source branch is `gh-pages`, folder `/`. Workflow completed on `main`. |
| Custom domain not verified | DNS `A`/`AAAA`/`www` CNAME; wait for propagation; [GitHub docs troubleshooting](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages). |
| HTTPS not available yet | Wait up to 24 hours after DNS is correct; then enable **Enforce HTTPS**. |
| Dependabot / Actions disabled | Repo and org **Actions** and **Dependabot** settings. |
| `npm run deploy` auth errors | `USE_SSH=true npm run deploy` or configure `git` credentials for HTTPS. |

---

## License / content

Add or update a license as appropriate for your organization. Marketing copy and assets live under `public/`.
