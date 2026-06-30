# How to Make GitHub Pages with Perplexity

A practical guide for building and deploying a static website to GitHub Pages using Perplexity as your AI coding and content assistant.

> **Real session note:** This guide was updated based on an actual session where Perplexity built and deployed [uutisseuranta.github.io](https://uutisseuranta.github.io) and configured the custom domain `uutisseuranta.net` — step by step, entirely through conversation.

---

## Overview

[GitHub Pages](https://pages.github.com/) is a free static site hosting service built into every GitHub repository. Combined with [Perplexity](https://www.perplexity.ai/), you can generate HTML, CSS, and JavaScript entirely through conversation — no local build tools required.

**What you need:**
- A GitHub account
- A public repository (or GitHub Pro/Team for private repos)
- A Perplexity account (free tier works)

---

## Step 1: Prepare Your Repository

If you don't have a repository yet, create one on GitHub. For a GitHub Pages site served directly at `https://username.github.io`, the repository **must be named `username.github.io`** (or `orgname.github.io` for an organization).

In this session, the repository [uutisseuranta/uutisseuranta.github.io](https://github.com/uutisseuranta/uutisseuranta.github.io) was used so the site is served at `https://uutisseuranta.github.io`.

Decide which deployment source you want:

| Source | Use case |
|---|---|
| `main` branch root (`/`) | Simple sites — all files in repo root |
| `main` branch `/docs` folder | Keeps source and site files separate |
| `gh-pages` branch | CI/CD workflows, generated static builds |

For a quick start, the `main` branch root is the easiest.

---

## Step 2: Generate Your Site with Perplexity

Open Perplexity and describe what you want to build. Be specific — the more context you give, the better the output.

**Example prompt used in this session:**

```
Make this into a github pages:
https://github.com/jaakkokorhonen/uutisseuranta
```

Perplexity fetched the repository, understood the project (a Finnish news aggregator), and generated a full single-file `index.html` with:
- Finnish-language copy and editorial design
- Sticky nav with SVG logo and dark/light mode toggle
- Hero section with animated pulse indicator and stats
- News feed in an asymmetric 2+1 editorial grid
- Features section with a live source activity bar chart widget
- Topics grid with 12 clickable topic chips
- Open-source CTA section
- Fully responsive mobile layout

**Key constraints to always mention to Perplexity:**
- "Single static HTML file" or "pure HTML/CSS/JS"
- "No server-side code"
- "Must work on GitHub Pages" (implies no Node.js backend)
- "CDN libraries only" if you want external dependencies
- Your preferred color scheme, language, or brand

---

## Step 3: Refine the Output

Perplexity generates code iteratively. After the first output:

1. **Test locally** — open the HTML file in your browser directly (`file://`)
2. **Identify issues** — broken layout, missing features, wrong colors
3. **Ask follow-up questions** — be specific:

```
The nav bar overlaps the hero section on mobile (375px width).
Fix the CSS so the nav stacks vertically below 768px.
```

Repeat until the result matches your vision. Perplexity maintains context within a conversation thread, so you can iterate without re-explaining the full project each time.

---

## Step 4: Push the File to GitHub

Perplexity can push files directly to GitHub using its MCP (Model Context Protocol) GitHub integration — no local git required.

In this session, after the first push attempt had a formatting issue, the retry worked by creating the file directly via the GitHub API:

```
try pushing again
```

Perplexity pushed `index.html` to `uutisseuranta/uutisseuranta.github.io` on the `main` branch in a single commit: [`03829e0`](https://github.com/uutisseuranta/uutisseuranta.github.io/commit/03829e07db364d2f2cd0fb8810437b0fbde41d2f).

### Via git CLI (alternative)

```bash
git clone https://github.com/uutisseuranta/uutisseuranta.github.io.git
cd uutisseuranta.github.io

cp ~/path/to/generated-site.html index.html

git add index.html
git commit -m "Initial GitHub Pages site"
git push origin main
```

### File naming

| File | Purpose |
|---|---|
| `index.html` | Root page — served at `https://username.github.io/` |
| `404.html` | Custom not-found page |
| `CNAME` | Custom domain (see Step 7) |

---

## Step 5: Enable GitHub Pages

> **GitHub Pages isn't enabled automatically — you need to turn it on in your repo settings.**

This is the one required manual step. Without it, your site will not be served even if `index.html` exists on the branch.

1. Go to [Settings → Pages](https://github.com/uutisseuranta/uutisseuranta.github.io/settings/pages) in your repo
2. Under **Source**, select **Deploy from a branch**
3. Choose branch: `main`, folder: `/ (root)`
4. Click **Save**

GitHub will build and deploy the site. The first deploy takes 1–3 minutes.

Your site URL will be:
```
https://uutisseuranta.github.io
```

---

## Step 6: Verify the Deployment

- Go to **Settings → Pages** — a green banner shows the live URL when deployment succeeds
- Check **Actions** tab for deployment logs if something fails
- Hard-refresh your browser (`Ctrl+Shift+R` / `Cmd+Shift+R`) to bypass cache

**Common issues:**

| Symptom | Cause | Fix |
|---|---|---|
| 404 on root | No `index.html` in deploy folder | Rename entry file to `index.html` |
| CSS/JS not loading | Absolute paths (`/style.css`) | Use relative paths (`./style.css`) |
| Blank white page | JS error on load | Open DevTools Console, fix the error |
| Old version showing | Browser or CDN cache | Hard-refresh, wait 5 min |
| Pages tab missing | Repo is private (free plan) | Make repo public or upgrade |

---

## Step 7: Add a Custom Domain (with Cloudflare)

In this session, `uutisseuranta.net` (managed in Cloudflare) was configured to point to the GitHub Pages site. The process has a specific order — doing it out of order breaks SSL provisioning.

### 7a. Add DNS records in Cloudflare

Go to **Cloudflare → uutisseuranta.net → DNS → Records** and add:

**Four A records** (apex domain → GitHub's IPs). Set **Proxy status = DNS only (grey cloud)**:

| Type | Name | Content |
|------|------|---------|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

**One CNAME record** for `www`:

| Type | Name | Content |
|------|------|---------|
| CNAME | www | `uutisseuranta.github.io` |

> ⚠️ The **grey cloud (DNS only)** is critical at this stage. If Cloudflare proxies traffic, GitHub cannot provision your SSL certificate and "Enforce HTTPS" stays greyed out.

### 7b. Push the CNAME file

Perplexity pushed a `CNAME` file to the repo root containing just:

```
uutisseuranta.net
```

Prompt used:
```
Push a file named CNAME (no extension) to the root of the main
branch containing just the domain: uutisseuranta.net
```

Perplexity discovered the file already existed with the correct content, so no change was needed.

### 7c. Set the custom domain in GitHub Pages settings

1. Go to **Settings → Pages** in your repo
2. Under **Custom domain**, type `uutisseuranta.net` and click **Save**
3. GitHub begins a DNS check — wait 5–30 minutes for propagation
4. Once the check passes, tick **Enforce HTTPS**

### 7d. Optionally re-enable Cloudflare proxy

After GitHub has issued the SSL certificate, you can enable the **orange cloud (Proxied)** on the A records in Cloudflare to get CDN, DDoS protection, and analytics. Set SSL/TLS mode to **Full (strict)** to avoid redirect loops.

---

## Step 8: Iterate with Perplexity

After the site is live, continue improving it through Perplexity conversations. Useful iteration patterns:

**Performance:**
```
The page has 3 external CDN scripts. Combine or lazy-load them
so the initial render is faster. No build tools.
```

**Accessibility:**
```
Audit this HTML for WCAG AA compliance. Check heading hierarchy,
alt text, color contrast, and keyboard navigation. Fix all issues.
```

**New features:**
```
Add a sticky header that hides on scroll down and reappears
on scroll up. Pure CSS/JS, no libraries.
```

**SEO:**
```
Add proper Open Graph meta tags, a <title>, and a <meta description>
for this news tracker site. Include Finnish-language og:locale.
```

---

## Tips for Better Perplexity Output

- **Paste your existing code** into the conversation when asking for changes — Perplexity has no memory of previous sessions by default
- **Specify the exact line or component** to change rather than asking for a full rewrite
- **Ask for explanations** alongside code: `"Explain the CSS Grid layout you used"` — this helps you maintain the code later
- **Version your files** in git before major changes so you can roll back
- **Use the file viewer** — paste generated HTML into [htmlpreview.github.io](https://htmlpreview.github.io/) for a quick preview before committing
- **Perplexity can push directly** — if you have the GitHub MCP integration enabled, you can skip local git entirely and ask Perplexity to push, update, or create files directly

---

## Actual Session Summary

This is what happened in the real session that produced this guide:

```
1. Prompt: "Make this into a github pages: https://github.com/jaakkokorhonen/uutisseuranta"
   → Perplexity generated full index.html (Finnish editorial landing page)

2. Prompt: "try pushing again"
   → Perplexity pushed index.html to uutisseuranta/uutisseuranta.github.io (main branch)
   → Commit: 03829e0

3. Manual step: Enable GitHub Pages in repo Settings → Pages
   → Site live at https://uutisseuranta.github.io

4. Prompt: "how do i configure uutisseuranta.net from cloudflare to work towards https://uutisseuranta.github.io"
   → Perplexity explained DNS A records, CNAME record, Cloudflare proxy settings

5. Prompt: "Push a file named CNAME (no extension) to the root of the main branch containing just the domain: uutisseuranta.net"
   → CNAME file already existed with correct content — no action needed

6. Manual step: Add DNS records in Cloudflare, set custom domain in GitHub Pages Settings
   → https://uutisseuranta.net live
```

---

## Resources

- [GitHub Pages documentation](https://docs.github.com/en/pages)
- [uutisseuranta.github.io repository](https://github.com/uutisseuranta/uutisseuranta.github.io)
- [Perplexity](https://www.perplexity.ai/)
- [HTML Preview for GitHub](https://htmlpreview.github.io/)

---

*Guide created and updated with Perplexity AI — June 2026*
