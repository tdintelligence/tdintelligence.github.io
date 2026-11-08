# TDACO Website — Project Context

Handoff document for continuing work on Tony Dang's personal website. Read this first.

## What this is

Personal website for **Tony Dang** — AI & Data Risk executive, Melbourne (Head of Data Risk & AI Governance, financial services). One-page site with a career section, a "Latest Thinkings" article section, and a contact section.

- **Live:** https://tdaco.au (GitHub Pages)
- **Local code:** `/Users/tonydang/AI/Agents/TDACO Website` — *add this folder to the session workspace*
- **GitHub:** `tdintelligence/tdintelligence.github.io`, branch `main`, site deploys from repo root
- **CNAME:** `tdaco.au`
- ⚠️ The project used to live in a Bionic session workspace as `my-website/`. It was moved to the folder above on 8 Nov 2026. The old path no longer exists.

## Tech stack

- Pure static site: hand-written HTML + CSS. **No frameworks, no build step, no dependencies.**
- Each page is self-contained (its own `<style>` block — the CSS template is duplicated per page).
- Only JS is a small inline IntersectionObserver script for scroll-reveal animations (gated behind `html.js`, respects `prefers-reduced-motion`).

## Design system (must be preserved)

Apple-style aesthetic. Do not deviate without asking.

- **CSS variables:** `--white #fbfbfd` · `--off-white #f5f5f7` · `--black #1d1d1f` · `--muted #6e6e73` · `--blue #0071e3` · `--blue-h #0077ed` · hairlines `rgba(0,0,0,.08)`
- **Type:** `-apple-system` stack; body 17px, `letter-spacing -0.022em`; h1 56px/700, h2 40px/600, h3 28px/600
- **Components:** cards 24px radius on off-white; pill buttons (`border-radius: 980px`); fixed blurred navbar (48–52px); eyebrow labels 13px uppercase `.06em` tracking
- **Images:** 3D renders, blue & white palette, glossy surfaces, studio lighting, light-blue background, **no text in image**
- **Breakpoint:** 734px (mobile)

## Site structure

```
index.html                     # hero → About → Career (4 cards) → Latest Thinkings → Connect → footer (+ subtle Admin link in footer)
writings/
  ai-governance-best-practices.html
  agentic-ai-control-framework.html
admin/
  index.html                     # password gate + viewer (self-contained; Web Crypto: PBKDF2 → AES-256-GCM)
  content.json                   # ENCRYPTED content payload (public file; only ciphertext is stored)
images/
  hero-1.png                   # in use (hero bg); hero-2..5.png are unused alternates
  experience-*.png             # 4 career card images
  article-*.png                # article covers/cards
CNAME · README.md
```

## How to add an article (recipe)

1. **Cover image:** generate via the `Comfyui-image` MCP server → `generate_image(prompt)`. Outputs 1024×1024 PNGs to `/Users/tonydang/AI/MCP/ImageGenerator/outputs/` (request read access to that folder). Style skeleton:
   `Clean 3D render illustration of {subject}: {details}, soft studio lighting, blue and white color palette, modern minimal technology aesthetic, smooth glossy surfaces, light blue background, high detail, no text, no words`
   Copy the result into `images/article-{slug}.png`.
2. **Article page:** copy `writings/ai-governance-best-practices.html` as the template. Keep the exact structure: `<style>` block → nav (logo `../index.html`, back-link `../index.html#writings`) → `.article-header` (date, h1, eyebrow) → `.article-cover` (16:9 img) → `.article-body` (h2 sections, h3 subsections, `ul`, `.pullquote`, `.key-points`, `.two-col` stat-boxes) → footer → signed `<em>— Tony Dang</em>`. Add meta description + theme-color in head.
3. **Index card:** add a `.writings-card` in the Latest Thinkings section (`.card-image` 16:10, date, h3, 1–2 line summary, `.card-link`). For unfinished articles use class `soon` + `.coming-soon-badge` (no link).

## Article voice & content rules

- Confident, plain-English, executive audience; practical over academic.
- Cite frameworks in prose (NIST, OWASP, MITRE ATLAS, AU Cyber Security Centre, Singapore IMDA, BSA) — no formal footnotes.
- Use real, current statistics with named sources in prose/stat-boxes.
- **Never include specifics in public copy:** no dollar figures, no named countries/markets (Melbourne location is fine), no named frameworks like Basel, no employer names.
- The section is called **"Latest Thinkings"** (deliberate — do not "correct" it to Writings).
- LinkedIn: `https://linkedin.com/in/dangdinhtrung`

## How to update the admin space (recipe)

The admin space is a **client-side-encrypted** private page at `admin/` (linked subtly from the index footer). No server, no framework — it fits the static-site stack.

- **How it works:** `admin/index.html` is a lock screen. The password is run through PBKDF2-HMAC-SHA256 (salt `tdaco-admin-salt-v1`, 600,000 iterations) to derive an AES-256-GCM key, which decrypts `admin/content.json` (base64 `iv` + `ct`, tag appended to ciphertext). The password is never stored or transmitted; the derived key is kept in `sessionStorage` for the tab's lifetime only. The published `content.json` is ciphertext — plaintext is never committed.
- **The admin password is NOT stored in this repo** (the repo is public). Ask the user for it whenever re-encryption is needed. If the password ever changes, re-encrypt `content.json` with the new one — the old ciphertext stops working.
- **To update content:**
  1. Edit the local plaintext file `admin/content-plain.json` (gitignored — never commit it). Schema: `{ "title", "updated", "entries": [ {"date", "title", "body"} ] }`.
  2. `body` uses light markdown: `# `/`## ` lines → headings, `- ` lines → bullets, other lines → text (all HTML-escaped by the viewer).
  3. Re-encrypt with PyCryptodome: `key = PBKDF2(password, b"tdaco-admin-salt-v1", dkLen=32, count=600000, hmac_hash_module=SHA256)`; `AES-GCM` with fresh 12-byte nonce; write `{"v":1, "iv": b64, "ct": b64(ciphertext+tag)}` to `admin/content.json`.
  4. Preview over HTTP (fetch() requires it), then commit + push on approval.
- **Security note for the user:** this is real encryption of the content (AES-256-GCM), stronger than a plain JS password gate — but it is client-side: the lock page is publicly reachable and the password must be known to the user. It is not a substitute for server-side auth; it hides the content, which is the goal for a personal admin space.
- **Content management (added 8 Nov 2026):** the admin page is a full client-side CMS — create / edit / delete entries, each with `created` + `updated` ISO timestamps, comma-separated **tags** (click a tag pill to filter), and **live search** (title + body + tags). Persistence is **`localStorage`** (per device/browser; keys `tdaco-admin-entries-v1` + `tdaco-admin-hidden-v1`), layered over the published seed: local entries with a seed's id override it; deleted seed ids are tracked in the hidden list so they don't resurrect. **Export** downloads all entries as plaintext JSON (`tdaco-admin-content-YYYY-MM-DD.json`, includes `hidden`); **Import** merges by id (import wins) with a confirm. `localStorage` is the portability story — tell the user to Export regularly.
- **Known gotchas (learned the hard way — do not reintroduce):** (1) the PBKDF2 key must be derived with `extractable: true`, otherwise `exportKey` throws `InvalidAccessError` and the unlock misreports as “Incorrect password”; (2) a stale session key (from an old password) must be discarded on decrypt failure and fall through to the typed password; (3) `storeKey` failure must never block a successful decrypt.

## Local preview (important quirk)

The in-app browser's `file://` preview **cannot load sibling-folder resources via `../`** (pages in `writings/` show broken images). Always preview over HTTP:

```
python3 -m http.server 8321 --bind 127.0.0.1 --directory "/Users/tonydang/AI/Agents/TDACO Website"
```

then open `http://localhost:8321/...` in the app browser.

## Workflow rules

- **Always preview first; commit + push only on explicit user approval** ("publish").
- After pushing, verify deployment: `curl -s -o /dev/null -w "%{http_code}" https://tdaco.au/{path}` (Pages rebuild takes ~1 min).
- Git identity is auto-configured (`tonydang@Tonys-MacBook-Pro.local`) — user is aware, left as-is.

## Current state (as of 8 Nov 2026)

- **Published:** 2 articles — *AI Governance Best Practices* (Nov 2026), *Agentic AI Control Framework* (Nov 2026).
- **Admin space:** `admin/` built 8 Nov 2026 (password-gated, client-side-encrypted content, CMS with tags/search/export-import).
- **Coming soon:** "AI Strategy & Roadmap" card (uses `images/article-strategy.png`, dimmed, badge).
- **Orphan asset:** `images/article-data.png` is unused (its topic was replaced by the agentic article) — can be deleted or repurposed.
- **Git:** clean, in sync with `origin/main` at commit `bdee654`.

## Suggested next work

1. Write the third article: **AI Strategy & Roadmap** (topic defined on the card; follow the recipe above).
2. Optional: per-article `og:image` tags for better social sharing cards.
3. Optional: delete orphaned `images/article-data.png` if the data topic is dropped for good.
