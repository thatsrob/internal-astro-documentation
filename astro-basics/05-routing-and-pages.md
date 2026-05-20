# 5. Routing and pages

Astro uses **file-based routing**: the folder structure under `src/pages/` defines your site’s URLs.

**Back to series:** [Astro basics index](./README.md) · **Previous:** [Slots and layouts](./04-slots-and-layouts.md)

---

## Basic rules

| File | URL (typical) |
|------|----------------|
| `src/pages/index.astro` | `/` |
| `src/pages/about.astro` | `/about/` |
| `src/pages/blog/index.astro` | `/blog/` |
| `src/pages/blog/law-firm-seo.astro` | `/blog/law-firm-seo/` |

- **Folders** become path segments.
- **`index.astro`** is the default file for a directory URL.
- There is **no central routes config** for standard pages—the filesystem is the route table.

Trailing slashes depend on `trailingSlash` in `astro.config.mjs`:

| Setting | Example URL |
|---------|----------------|
| `'always'` (common default) | `/about/` |
| `'never'` | `/about` |
| `'ignore'` | Astro accepts both; host may normalize |

Our site uses directory-style URLs (`/about/`). Internal links should match that style so you do not create duplicate URLs.

### Non-`.astro` pages

Astro also supports:

| File | Notes |
|------|-------|
| `page.md` / `page.mdx` | Markdown routes (need MDX integration for `.mdx`) |
| `page.html` | Raw HTML route |

Constellation uses **`.astro` only** for production pages—HTML bodies live inside templates, not separate `.md` files.

---

## Only `src/pages/` creates routes

Files outside `src/pages/` are components only:

```
src/
  components/     → no URL
  layouts/        → no URL
  pages/          → URLs live here
```

Import components from anywhere; only pages are addressable in the browser.

---

## Dynamic routes

For paths that depend on data (e.g. one template, many slugs), use **brackets** in the filename:

| File | Matches |
|------|---------|
| `src/pages/blog/[slug].astro` | `/blog/anything/` |
| `src/pages/city/[city]/[service].astro` | `/city/austin/seo/` |

In frontmatter, read params:

```astro
---
export async function getStaticPaths() {
  return [
    { params: { slug: 'law-firm-seo' } },
    { params: { slug: 'ppc-guide' } },
  ];
}
const { slug } = Astro.params;
---
```

`getStaticPaths()` tells Astro which URLs to build in **static** mode.

**Constellation today:** most routes are **explicit files** (`law-firm-seo.astro`), not dynamic `[slug].astro`, because migration generated one file per WordPress URL. Dynamic routes are optional for future refactors.

### Rest parameters (catch-all)

`src/pages/docs/[...path].astro` matches `/docs/a`, `/docs/a/b`, etc. `Astro.params.path` is a string like `a/b`. Useful for nested doc trees; we do not use this pattern on the marketing site today.

---

## `getStaticPaths` and data

For static sites, every dynamic URL must be known at **build time**:

```astro
---
export async function getStaticPaths() {
  const posts = await loadPostsFromSomewhere();
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}
const { post } = Astro.props;
---
```

If a path is missing from `getStaticPaths`, it will not exist in `dist/` (404 in production).

---

## Redirects

Configure redirects in **`astro.config.mjs`** (build-time) or at the host (Cloudflare, Netlify):

```js
export default defineConfig({
  redirects: {
    '/old-blog-slug/': '/blog/new-slug/',
  },
});
```

Our site uses config redirects for legacy WordPress paths. See [REDIRECTS-AND-URLS.md](../REDIRECTS-AND-URLS.md).

---

## 404 pages

Add `src/pages/404.astro` for a custom not-found page. Static hosts must be configured to serve it for unknown paths (Cloudflare Pages supports this).

---

## `public/` URLs (not routes)

Files in `public/` are copied to the site root **without** processing:

| File | URL |
|------|-----|
| `public/images/logo.svg` | `/images/logo.svg` |
| `public/robots.txt` | `/robots.txt` |

Use `public/` for assets that must keep exact paths and are not imported from components.

---

## Linking between pages

Use normal HTML paths (root-relative):

```astro
<a href="/book-call/">Book a call</a>
<a href="/blog/law-firm-seo/">Read the guide</a>
```

Astro does not require a special `<Link>` component for static sites.

---

## Three URLs to keep straight (migration context)

When moving from WordPress, align:

| Concept | Question |
|---------|----------|
| **Production URL** | What ranks today? |
| **Astro file path** | What path does `src/pages/...` produce? |
| **Canonical** | What should search engines index? |

If file path ≠ production URL, you need a **redirect**, not only a canonical tag. See [ROUTING-REFERENCE.md](../ROUTING-REFERENCE.md).

### Finding routes in the repo

```bash
# List all page files
find src/pages -name '*.astro' | head

# Resolve which file owns a URL fragment
find src/pages -name '*law-firm-seo*'

# See all canonicals pointing at production
grep -r 'canonicalUrl=' src/pages --include='*.astro' | head
```

### `site` in config

`astro.config.mjs` sets `site: 'https://www.goconstellation.com'` so Astro can build absolute URLs (sitemaps, RSS, some meta helpers). Page-level `canonicalUrl` props still override per-page SEO when you set them explicitly.

---

## Next

[6. Build time vs runtime →](./06-build-time-vs-runtime.md)
