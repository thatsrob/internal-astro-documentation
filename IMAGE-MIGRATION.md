# Image migration

How to move images off the live WordPress server (`wp-content/uploads`) and into this repo so the Astro site does not depend on WordPress staying online.

Production today serves years of media from `https://goconstellation.com/wp-content/uploads/...`. Many Astro pages still point at those URLs. That works while WordPress is up; after cutover or shutdown, those links break unless you migrate the files.

---

## Why this matters

The static site is hosted on Cloudflare Pages. It does not include WordPress’s upload library. Any `src` or CSS `url()` that still points at `wp-content` is a **remote dependency** on the old stack.

You want images to live under `public/images/` (or another CDN you control) and be referenced as `/images/...` so they ship inside `dist/` on every deploy.

---

## What’s in the repo today

### Local images (good)

Most city pages, case studies, homepage Refore assets, logos, and OG images already use paths like:

- `/images/attorney-marketing-austin-title.webp`
- `/images/case-study/...`
- `/images/constellation-og.jpg`
- `/images/refore/...`

`scripts/build_city_pages.py` downloads and converts many city heroes to WebP when you run it. That path is the model for “images owned by the repo.”

### Still hotlinking WordPress (needs work)

As of the last audit, **12 `.astro` files** under `src/` still reference `wp-content/uploads`, plus:

- `src/layouts/templates/CityTemplate.astro` — case-study card thumbnails in a JSON-like array (six images)
- `src/pages/book-call.astro` — testimonial headshots (four photos)
- `src/layouts/templates/HomepageTemplate-v2.astro` — badges and practice-area cards (v2 is not production homepage, but still in repo)
- `src/styles/divi-theme-options.css` — logo via `content: url(...)` (sitewide if that CSS is loaded)

**Blog posts with inline hotlinks** (examples): `law-firm-names`, `law-firm-letterhead`, `best-fonts-legal-documents`, `martindale-hubbell-lawyer-directory`, `findlaw-lawyer-directory`, `catchy-law-firm-names`, `is-avvo-legit`, `law-firm-seo/local-seo-for-lawyers`.

Find the current list anytime:

```bash
grep -r "wp-content/uploads" src/ --include="*.astro" -l
grep -r "wp-content/uploads" src/ --include="*.css" -l
```

---

## Priority order

Migrate in this order so launch blockers clear first.

| Priority | Where | Why |
|----------|--------|-----|
| **P0** | `book-call.astro` | Primary conversion page; testimonial photos |
| **P0** | `CityTemplate.astro` | Case-study cards on every city landing page |
| **P1** | Top-traffic blog posts (from Search Console) | SEO and trust content |
| **P1** | `law-firm-seo/local-seo-for-lawyers.astro` | Long guide, multiple images |
| **P2** | Remaining blog inline images | Fix when touching that post for QA |
| **P2** | `HomepageTemplate-v2.astro`, `motion-extracted` CSS | Only if still referenced in build |

---

## How to migrate one image (manual)

This is the default workflow for a single asset.

### 1. Get the file

Open the production URL in a browser, or download from WordPress admin / a full `uploads` backup if you have one.

Example production URL:

`https://goconstellation.com/wp-content/uploads/2024/12/Rick-Waltman-min.png`

### 2. Put it in `public/images/`

Use a clear folder structure:

```
public/images/
├── wp-uploads/              # optional mirror of WP year/month
│   └── 2024/12/
│       └── Rick-Waltman-min.webp
├── case-study/              # case study heroes
├── book-call/               # testimonial photos (suggested)
└── attorney-marketing-…     # city assets (often already here)
```

**Format:** Prefer **WebP** for photos (smaller, faster). PNG is fine for logos with transparency. Keep filenames lowercase and hyphenated.

**Resize:** Hero widths above ~1600px are usually unnecessary; city script caps around 1600px width. Testimonials can be ~200–400px wide.

### 3. Update the reference

Change the URL in the `.astro` file (or template) from:

```html
src="https://goconstellation.com/wp-content/uploads/2024/12/Rick-Waltman-min.png"
```

to:

```html
src="/images/book-call/rick-waltman-min.webp"
```

For props in JavaScript objects (e.g. `book-call.astro` testimonials), update the `photo` field the same way.

### 4. Fix lazy-load leftovers

Scraped WordPress HTML often has:

- `src` pointing at a tiny SVG placeholder
- `data-lazy-src="https://goconstellation.com/wp-content/..."`

For migrated posts, set **`src`** to the real `/images/...` path and remove `data-lazy-src` unless you add a real lazy-load script (Astro does not ship WordPress’s lazy loader).

### 5. Verify

```bash
npm run dev
```

Open the page, confirm images load with the network tab showing your staging host (not `goconstellation.com/wp-content`). Run `npm run build` to ensure nothing breaks.

---

## Migrating many blog images

There is no dedicated “migrate all images” script in this repo. Practical approaches:

1. **Per post during QA** — When you review a blog row on the tracker, fix its images in the same PR.
2. **Batch download** — If you have a WordPress `uploads` folder backup, copy files into `public/images/wp-uploads/` preserving year/month, then run a careful find-replace on `src/pages/blog/` (review diffs; do not blind-replace across the whole repo).
3. **Re-scrape** — `build_astro_post.py` **removes** images from the body; use only if you plan to re-add images manually afterward.

---

## City template case-study cards

`CityTemplate.astro` embeds six case-study teasers with hardcoded production URLs for both **links** and **images**. For image migration:

1. Download each `.../wp-content/uploads/2024/03/*.jpg` (or current path).
2. Save under `public/images/case-study/` or `public/images/wp-uploads/2024/03/`.
3. Change each `img:` value in the template data to `/images/...`.

Consider updating `url:` values to Astro paths (`/case-study/sabbeth-law/`) in the same pass—that is a routing concern, but often done together.

---

## Book-call testimonials

`book-call.astro` stores four client photos as full WordPress URLs in a `testimonials` array. Create `public/images/book-call/` (or similar), migrate the four WebP/PNG files, and point `photo` at `/images/book-call/...`.

These are high visibility on the money page—do them before launch.

---

## CSS background images

If an image is referenced only in CSS (e.g. `divi-theme-options.css` with `content: url(https://goconstellation.com/wp-content/...)`), either:

- Download the asset to `public/images/` and change to `url(/images/...)`, or
- Stop loading that CSS file on production if it is legacy and unused.

Check whether `production-extracted.css` or `BaseLayout` still pulls in the file that references WordPress.

---

## After migration

- Re-run the grep commands above; the list should be empty (or only acceptable exceptions documented).
- Spot-check on **staging** with WordPress still up, then again after cutover.
- Keep a **full `uploads` backup** off the server even after WP shutdown (legal/historical reference)—see WordPress decommission docs.

---

## Quick checklist (one page)

- [ ] Listed all `wp-content` URLs on that page
- [ ] Files saved under `public/images/`
- [ ] References updated to `/images/...`
- [ ] Lazy-load placeholders fixed
- [ ] Meaningful `alt` text present
- [ ] Page checked in dev and build passes
