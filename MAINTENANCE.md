# Spot Vision website — update & maintenance guide

Everything you need to change, preview, and publish **spotvision.io**.

---

## 1. What this site is (and where it lives)

- **One static file.** The entire site is `public/index.html` — self-contained: inline CSS, inline SVG (chart + contour art + icons), fonts pulled from Google Fonts. **No build step, no framework.** Edit the file, deploy, done.
- **Repo:** `github.com/spotvision/website` (this repo). It is the source of truth — always commit + push after you deploy.
- **Host:** Firebase Hosting.
  - **Firebase project:** `spotvision-patterns2`
  - **Production site** (serves `spotvision.io` + `www`): site id **`spotvision-patterns2`**
  - **Preview/staging site:** **`spotvision-landing`** → `https://spotvision-landing.web.app`
  - **Old "Design Tagger" site** (preserved): **`spotvision-legacy`** → `https://spotvision-legacy.web.app`
- **Domain:** `spotvision.io` is registered at **GoDaddy**; its DNS is hosted at GoDaddy and points the apex at Firebase. (You own it.)

> Note: this website repo is intentionally **separate** from the SpotNet research monorepo — keep it that way. The research repo holds confidential/patent material and must never be public.

---

## 2. One-time setup (on a new machine)

```bash
npm install -g firebase-tools      # the Firebase CLI
firebase login                     # Google account with access to spotvision-patterns2
git clone git@github.com:spotvision/website.git
cd website
```

- `firebase login` opens a browser. **On a remote/headless box** where the browser can't reach it, use `firebase login --no-localhost` and paste the code back.
- Credentials cache locally (`~/.config/configstore/firebase-tools.json`) — you never paste a password anywhere else.

---

## 3. Make a change

Edit **`public/index.html`**. The page is organized as labelled sections you can search for:

| Section (`id`) | What it is |
|---|---|
| hero | headline, sub, the 4 stat chips, CTAs |
| `#case` | "Why sensing" — 6 benefit cards + the sensor-shipments SVG chart |
| `#cost` | "The resource bill" — electricity / water / bandwidth / hardware / batteries |
| `#solution` | "The fabric" — the µJ/mJ/J tiers + comparison table + gains |
| `#how` | "How it works" — 5 properties + the `<7 µJ` measured proof card |
| `#packaging` | "It ships as an IP core" — 4 steps + Gate/Cascade/Pipeline + modality chips |
| `#contact` | closing CTA band + resources/money/product summary + email |

- **Colors** are CSS variables at the top of `<style>` in `:root` (`--paper`, `--forest`, `--power` = amber, `--water`, `--cell` = terracotta, `--ink`, …). Change them there and the whole site re-tones.
- **Fonts:** IBM Plex Sans / Condensed / Mono, loaded via the `<link>` in `<head>`.
- **Contact email:** `info@spotvision.io` (search for it; it's in the CTA button and footer as `mailto:` links).

Preview by opening `public/index.html` in a browser, or deploy to the preview site (below).

---

## 4. Deploy

**Production (goes live at spotvision.io):**
```bash
firebase deploy --only hosting --project spotvision-patterns2
```
`firebase.json` already points `hosting.site` at `spotvision-patterns2`, so this publishes to the live domain. Live within ~1 minute. **Then commit + push:**
```bash
git add -A && git commit -m "…" && git push
```

**Preview first (staging, without touching live):** deploy the same `public/` to the `spotvision-landing` site — either temporarily set `hosting.site` to `spotvision-landing` and deploy, or create a sibling `firebase.json` with `"site": "spotvision-landing"`. It appears at `https://spotvision-landing.web.app` and does not affect `spotvision.io`.

---

## 5. Rollback

- **Fastest:** Firebase console → Hosting → the site → **release history → "Rollback"** to any prior version.
- Or `firebase hosting:clone spotvision-patterns2:live spotvision-patterns2:live --version <id>`.
- Or `git revert <commit>` then redeploy.

---

## 6. Domain / DNS (already configured — reference only)

- `spotvision.io` apex points at Firebase; the custom domain is attached to the `spotvision-patterns2` site in the Firebase console. `www` follows.
- **The Firebase CLI cannot manage custom domains** — domain changes are done in the **Firebase console** (Hosting → Add custom domain) + **GoDaddy DNS** (Domain Portfolio → spotvision.io → Manage DNS).
- The old site is preserved and live at `spotvision-legacy.web.app`. To give it `old.spotvision.io`: Firebase console → `spotvision-legacy` site → Add custom domain → `old.spotvision.io`, then add the DNS record it shows you in GoDaddy.

---

## 7. Content & disclosure rules (READ before publishing anything)

This is a **public, crawlable** site, so it is held to a higher bar than an internal deck:

1. **Hold the unfiled "how."** Public framing only — "multiplier-free / zero-DSP", decide-early, flash-resident, measured energy on named public datasets. Do **not** expose internal architecture mechanisms.
2. **Every hard number needs a credible, cited source.** Prefer a real *measured* figure (with its year) or a clearly-labelled *projection*. No unsourced numbers presented as fact. The resource-bill figures cite IEA / arXiv / UN and are framed as third-party estimates.
3. **Keep the measured-vs-target hedges.** The `<7 µJ` is an *upper bound* on a CIFAR-10-grayscale workload (not VWW); VWW accuracy/energy are "in characterisation." Don't quietly upgrade a target into a claim.
4. **Don't overstate the offering.** We do the training + compilation (it's not self-serve, and it doesn't just run a customer's model). It's licensable IP for **custom silicon** — SoC / ASIC / sensor die — **not** a drop-in for an off-the-shelf MCU.
5. When in doubt, re-scan a change for anything "how"-level before it goes public — foreign patent filings have no grace period.

---

## 8. Gotchas we already hit (don't repeat them)

- **Don't apply an SVG `filter:url(#id)` to an `<img>`.** Chrome hides the element if the filter doesn't resolve and Safari renders it blank — images vanish. Use CSS (`filter:grayscale(...)` + blend-mode overlays) for effects like duotone.
- **Hard-refresh** (Cmd/Ctrl-Shift-R) after a deploy — the CDN + browser cache the old page for a bit.
- **Cache-stable years:** the resource figures are written present-tense ("already draw ~X") with only *projection* years, so they don't read as stale as time passes.

---

## 9. Quick reference

```bash
# edit
$EDITOR public/index.html

# publish to spotvision.io
firebase deploy --only hosting --project spotvision-patterns2
git add -A && git commit -m "site: <what changed>" && git push

# preview only (no live change): deploy public/ to site "spotvision-landing"
```

| Thing | Value |
|---|---|
| Live domain | spotvision.io (+ www) |
| Firebase project | spotvision-patterns2 |
| Prod site id | spotvision-patterns2 |
| Preview site | spotvision-landing.web.app |
| Old site | spotvision-legacy.web.app |
| Registrar / DNS | GoDaddy |
| Contact email | info@spotvision.io |
| Source repo | github.com/spotvision/website |
