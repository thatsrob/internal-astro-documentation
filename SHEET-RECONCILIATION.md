# Sheet reconciliation

How to align the **Google migration tracker** (325 URLs, Done/Remaining columns) with what is actually in the **Git repo** (~386 routes). Without this step, progress percentages mislead the team: the sheet may say “114 cities left” while the repo already has ~121 city files.

This is a **process** doc, not a script. Run it once after exporting sheet tabs, then again before launch sign-off.

---

## Why the numbers do not match

Three different “counts” float around:

| Source | Typical number | What it measures |
|--------|----------------|------------------|
| Migration tracker summary | **325** URLs | Manifest the team agreed to migrate |
| Tracker “Done” | **172** | Rows marked complete on the sheet |
| Repo `src/pages/**/*.astro` | **~396** files | Every route Astro builds |
| Repo excluding `/examples/` | **~386** | Production-intent pages |

**325 vs 386** — The repo has **more** than the manifest: nested guides (`law-firm-seo/*`), extra cities, hub pages (`blog/index`, `case-studies`), partner pages, and routes never added as sheet rows.

**172 Done vs “feels 80% built”** — “Done” might mean QA signed off, while Git only means a file exists. Or the opposite: file exists but the sheet was never updated.

Until everyone agrees what **Done** means, the summary row is approximate.

---

## Step 0 — Agree what “Done” means

Get one definition from whoever owns the tracker (e.g. Patrick). Write it in the sheet README row or a pinned comment.

**Suggested definitions (pick one per column or one global):**

| Status | Meaning |
|--------|---------|
| **Not started** | No `.astro` file; production still WordPress only |
| **In repo** | File exists on `main`; not reviewed on staging |
| **Staging QA** | Reviewed on staging; issues logged or fixed |
| **Done** | Staging QA pass; canonical/redirects OK for that URL; ready for launch batch |

Avoid marking **Done** when only “we ran the scraper once.” That is **In repo**.

---

## Step 1 — Export the sheet to CSV

In Google Sheets, for each tab:

**File → Download → Comma-separated values (.csv)**

Save into `documents/data/` with consistent names:

| Tab | Filename |
|-----|----------|
| Blog Posts | `blog-posts.csv` |
| City / Location | `cities.csv` |
| Case Studies | `case-studies.csv` |
| Podcast Episodes | `podcasts.csv` |
| Practice Areas | `practice-areas.csv` |
| Service Pages | `services.csv` |
| Core Pages | `core-pages.csv` |
| Generic / Other | `generic.csv` |

Commit CSVs to Git so the team reconciles against the same snapshot (refresh after big sheet updates).

You already have **Page Types** as `migration-tracker-page-types.csv` — that tab is architecture reference, not per-URL status.

---

## Step 2 — Normalize URLs for comparison

Sheet URLs and repo paths need the same shape before you compare.

**Rules:**

1. Use lowercase paths.
2. Strip scheme and host: `https://www.goconstellation.com/foo/` → `/foo/`
3. Ensure **trailing slash** on paths (site convention).
4. Strip query strings and hashes.

**Production URL → expected Astro file (common patterns):**

| Production path | Likely file |
|-----------------|-------------|
| `/` | `src/pages/index.astro` |
| `/blog/post-slug/` | `src/pages/blog/post-slug.astro` |
| `/post-slug/` (root blog on WP) | Often `src/pages/blog/post-slug.astro` with canonical still at `/post-slug/` |
| `/attorney-marketing-austin/` | `src/pages/attorney-marketing-austin.astro` |
| `/law-firm-marketing-dallas/` | `src/pages/law-firm-marketing-dallas.astro` |
| `/case-study/kairos-law-group/` | `src/pages/case-study/kairos-law-group.astro` |
| `/kairos-law-group-40xroi-case-study/` (old WP) | May be canonical only; file under `/case-study/` |
| `/podcasts/episode-slug/` | `src/pages/podcasts/episode-slug.astro` |
| `/family-law-marketing/` | `src/pages/family-law-marketing.astro` |

When production path ≠ file path, the row is still **reconciled** if the file exists **and** redirect/canonical plan is documented—not “missing.”

---

## Step 3 — Build a repo inventory

From repo root:

```bash
# List all page files (excluding examples)
find src/pages -name '*.astro' ! -path '*/examples/*' | sort > /tmp/astro-routes.txt

# Count by folder
find src/pages/blog -name '*.astro' ! -name 'index.astro' | wc -l
find src/pages -maxdepth 1 -name 'attorney-marketing-*.astro' | wc -l
find src/pages -maxdepth 1 -name 'law-firm-marketing-*.astro' | wc -l
find src/pages/case-study -name '*.astro' | wc -l
find src/pages/podcasts -name '*.astro' ! -name 'index.astro' | wc -l
```

Optional: read `canonicalUrl` from each file to map “SEO URL” vs “file URL”:

```bash
grep -rh 'canonicalUrl="https://goconstellation.com' src/pages --include='*.astro' | head -20
```

That helps explain rows where the sheet URL is root but the file is under `/blog/`.

---

## Step 4 — Reconcile tab by tab

For each CSV export, add or use columns:

| Column | Purpose |
|--------|---------|
| `production_url` | From sheet |
| `slug` | Derived path segment |
| `astro_file` | Path if found, else blank |
| `repo_status` | `missing` / `exists` / `canonical_mismatch` |
| `sheet_status` | Done / Remaining from sheet |
| `recommended_action` | `mark_done` / `qa_only` / `build` / `redirect_note` |

### Blog Posts (~132 rows)

**Expectation:** ~160 files in `src/pages/blog/` but only 132 on sheet.

| Situation | Action |
|---------|--------|
| Sheet Remaining, file exists | QA on staging; if pass → mark Done |
| Sheet Done, no file | Find URL typo or build page |
| File exists, not on sheet | Add row or accept out-of-manifest; document in reconcile notes |
| Canonical at root, file in `/blog/` | Not “missing”—track redirect for launch |

### City / Location (~118 rows)

**Expectation:** Sheet may show **4 Done / 114 Remaining** while repo has **~121** city files.

| Situation | Action |
|---------|--------|
| File exists, sheet Remaining | Almost always **mark Done** after spot-check, not rebuild |
| Sheet Done, file missing | Run `build_city_pages.py` for that slug |
| Wrong slug (`atlanta` vs `atlanta-ga`) | Fix sheet or rename file to match production URL |

### Case Studies (~29 rows)

Match **production slug** (often long) to **`/case-study/<short>/`** file. Reconcile redirects separately.

### Podcasts (~22 rows)

Sheet may show **100% Done**. Confirm 24 files under `podcasts/` and spot-check embeds.

### Practice Areas (~9 rows)

Root-level `*-marketing.astro` files. Three “remaining” on sheet = build or QA gap.

### Service Pages (~4 on sheet tab)

Repo has more service-like pages (`meta-ads`, flagships) than the four manifest rows. List **sheet URLs** vs **repo** explicitly; missing flagship URLs are real build work.

### Core + Generic (~10 rows)

Small set; often hand-built. Generic tab at **0%** is a real gap if those four URLs are still WordPress-only.

---

## Step 5 — Classify every mismatch

Use a short code in your notes:

| Code | Meaning |
|------|---------|
| **A** | Sheet behind Git → update sheet to Done after QA |
| **B** | Git behind sheet → build or migrate URL |
| **C** | Path mismatch only → file OK; add redirect/canonical task |
| **D** | In repo but out of manifest → decide add to sheet or ignore |
| **E** | Duplicate or deprecated → remove or 301 |

---

## Step 6 — Update team artifacts

After reconciliation:

1. **Sheet** — Update Done/Remaining counts; fix summary row if the sheet formulas allow.
2. **`linear-progress.md`** — Refresh ASCII bars from new summary (manual).
3. **`TODO.md`** — Replace “25 blogs remaining” bullets with **named URLs** from CSV where possible.
4. **`PROGRESS.md`** — Update repo vs sheet table with reconciliation date.

You do not have to reconcile all 325 rows in one day. Order:

1. Generic + Core (small, launch blockers)
2. Service tab + flagships
3. Case studies (3 remaining)
4. Blogs marked Remaining
5. Cities (batch: mark exists + QA sample)

---

## Sanity checks after reconciliation

- [ ] Summary **Done + Remaining = 325** (or manifest total documented if rows added/removed)
- [ ] No duplicate `.astro` files for the same production URL
- [ ] Every **Remaining** row has either an `astro_file` or an assigned owner
- [ ] Canonical mismatches logged for launch redirects
- [ ] Patrick (or owner) signed off on **Done** definition

---

## Example row outcomes

**Blog:** Sheet says Remaining for `/law-firm-seo/`. File `src/pages/blog/law-firm-seo.astro` exists. Canonical points to `https://goconstellation.com/law-firm-seo/`.  
→ **Code A + C.** QA pass → mark Done; add redirect `/law-firm-seo/` → `/blog/law-firm-seo/` or reverse per SEO decision.

**City:** Sheet says Remaining for `/attorney-marketing-denver/`. File exists. Hero uses local WebP.  
→ **Code A.** Spot-check staging → mark Done.

**Service:** Sheet lists `/legal-marketing-services/`. No matching `.astro` at that path.  
→ **Code B.** Build page or 301 to `/services/`; until then stays Remaining.

---

## When to re-run reconciliation

- After a large merge batch (50+ pages)
- Before Phase 4 sign-off
- Before launch week (final manifest vs `dist/` sitemap)
- After Patrick changes sheet structure or Done rules

---

## What this doc does not do

It does not replace the migration tracker (still the scope authority). It does not auto-sync sheet ↔ Git—no script in repo does that today. If you add a script later, keep this doc as the human-readable rules for slug matching and status definitions.
