# EventSeek — Deployment Guide

## Deploy to `zhiyiarchi.com` via Vercel + GitHub

### Pre-flight Checklist

| Item | Status |
|------|--------|
| Single HTML file, no build step | ✅ |
| All data embedded inline (~96 KB) | ✅ |
| Zero `fetch()` calls at runtime | ✅ |
| Leaflet loaded from CDN with SRI hashes | ✅ |
| Map tiles from CartoDB (free tier) | ✅ |
| Mobile-first, 430px max-width | ✅ |
| 8 screens, 13 core functions | ✅ |
| 204 plots embedded (20 curated + 185 background) | ✅ |

### Project Structure

```
website/                  ← This is your deploy root
├── .gitignore            ← Git exclusions
├── vercel.json           ← Vercel config (clean URLs, CORS)
├── index.html            ← Production site (self-contained)
├── prototype.html        ← Earlier prototype (can delete pre-deploy)
└── data/                 ← Source derivation files (not loaded at runtime)
    ├── all_study_plots.json
    ├── curated_plots.json
    └── typo_summary.json
```

> **Note:** The `data/` JSON files are source artifacts — `index.html` embeds all plot data inline as JavaScript arrays. They are safe to deploy (small files) but not required for the site to work.

---

## Step 1 — Initialize Git in the `website/` Folder

Open a terminal in the `website/` directory:

```bash
cd website
git init
git add -A
git commit -m "EventSeek — production v1

Urban Search Engine for Matching Events & Spatial Conditions.
Self-contained single-file site with embedded 185-plot dataset.

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

## Step 2 — Push to GitHub

### 2a. Create a new GitHub repository

1. Go to https://github.com/new
2. Repository name: `eventseek` (or `zhiyiarchi` — your choice)
3. Keep it **Public** (Vercel can deploy private repos too, but public is fine for a portfolio project)
4. Do NOT initialize with README, .gitignore, or license (you already have files)
5. Click "Create repository"

### 2b. Connect and push

```bash
git remote add origin https://github.com/YOUR_USERNAME/eventseek.git
git branch -M main
git push -u origin main
```

---

## Step 3 — Deploy on Vercel

### 3a. Import the GitHub repo

1. Go to https://vercel.com and log in (use "Continue with GitHub")
2. Click **"Add New…"** → **"Project"**
3. Find and select your `eventseek` repo
4. Vercel auto-detects the framework — **no configuration changes needed**:
   - Framework Preset: **Other** (or leave as detected)
   - Build Command: (leave empty — no build step)
   - Output Directory: (leave empty — root is the deploy)
   - Root Directory: (leave as default — `./`)
5. Click **"Deploy"**

### 3b. Verify the deployment

Vercel will give you a preview URL like `eventseek-abc123.vercel.app`. Open it and:
- Type "night market" → should show searching animation
- Tap through to the map → 3-level reveal should animate
- Tap back and re-enter map → should show final state immediately
- Test all 8 events on the landing page

---

## Step 4 — Add Custom Domain `zhiyiarchi.com`

### 4a. In Vercel Dashboard

1. Go to your project → **Settings** → **Domains**
2. Enter `zhiyiarchi.com` and click **"Add"**
3. Vercel will recommend adding `www.zhiyiarchi.com` as well (redirect to apex) — accept this
4. Vercel shows the DNS records you need to add

### 4b. At Your Domain Registrar

Log in to wherever `zhiyiarchi.com` is registered (Namecheap, GoDaddy, Cloudflare, etc.) and add these DNS records:

**For the apex domain** (`zhiyiarchi.com`):
```
Type:  A
Name:  @
Value: 76.76.21.21
```
> If your registrar doesn't support CNAME flattening for apex domains, use this A record pointing to Vercel's IP.

**For the www subdomain** (recommended):
```
Type:  CNAME
Name:  www
Value: cname.vercel-dns.com
```

### 4c. Wait for DNS Propagation

- DNS changes can take 1–10 minutes (sometimes up to 48 hours)
- Vercel will auto-provision an SSL certificate via Let's Encrypt once DNS resolves
- The domain status in Vercel will change from "Configuring" → "Valid" when ready

### 4d. Automatic www → apex redirect

Vercel handles this automatically. When a visitor goes to `www.zhiyiarchi.com`, they'll be redirected to `https://zhiyiarchi.com`.

---

## Step 5 — Post-Deployment Verification

Open `https://zhiyiarchi.com` and verify:

- [ ] Landing page loads with 8 event cards
- [ ] Search animation shows dynamic counts ("Scanning 185 candidate spaces…")
- [ ] Match reveal shows typology + count annotation
- [ ] Map renders with Leaflet tiles
- [ ] Map 3-level progressive reveal animates correctly
- [ ] Counter badge transitions: 185 → {N} typology → 4
- [ ] Back-navigation shows final map state instantly
- [ ] All 5 different typologies produce correct counts
- [ ] SSL padlock shows in browser address bar
- [ ] Mobile layout looks correct (test on phone)
- [ ] Share links preserve state

---

## Optional — Remove `prototype.html`

If you don't want the prototype publicly accessible:

```bash
git rm prototype.html
git commit -m "Remove prototype from production deploy"
git push
```

Vercel auto-redeploys on push.

---

## Optional — Remove `data/` JSON Files

These 3 files (`data/*.json`) are source derivation files. The production site embeds all data inline and never loads them. If you want a cleaner deploy:

```bash
git rm -r data/
git commit -m "Remove source data files (data is embedded inline in index.html)"
git push
```

---

## Rollback

If something breaks:
1. In Vercel dashboard → **Deployments** tab
2. Find the last working deployment
3. Click **"…"** → **"Promote to Production"**

---

## Local Testing Before Push

To test locally with the exact Vercel behavior:

```bash
cd website
npx vercel dev
```

Or simply open `index.html` in a browser — it's a static file, no server required.

---

## File That Never Changes

[vercel.json](vercel.json) — Clean URL support + long cache headers for the `data/` folder.
