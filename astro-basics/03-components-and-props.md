# 3. Components and props

In Astro, **every `.astro` file is a component**. You compose pages by importing components and using them like custom HTML tags.

**Back to series:** [Astro basics index](./README.md) · **Previous:** [The `.astro` file](./02-astro-file-syntax.md)

---

## Importing and using a component

```astro
---
import SiteHeader from '../components/SiteHeader.astro';
---
<SiteHeader />
```

- Use the **default export** pattern: `import Name from './Name.astro'`.
- Tag name matches the import name (PascalCase convention).
- Paths are relative to the current file unless you configure aliases in `tsconfig.json`.

---

## Passing props

Props are **attributes** on the component tag:

```astro
<SiteHeader activePath="/blog/" showCta={true} />
```

In the child component, read props from **`Astro.props`**:

```astro
---
// SiteHeader.astro
const { activePath = '/', showCta = false } = Astro.props;
---
<nav>
  <a href="/" class:list={[{ active: activePath === '/' }]}>Home</a>
  {showCta && <a href="/book-call/">Book a call</a>}
</nav>
```

| Concept | Astro equivalent |
|---------|------------------|
| React `props.children` | **Default slot** (content between tags) |
| React prop types | TypeScript `interface Props` (optional but recommended) |

---

## TypeScript props (recommended)

```astro
---
interface Props {
  title: string;
  description?: string;
  count?: number;
}
const { title, description = '', count = 0 } = Astro.props;
---
<h1>{title}</h1>
```

- Optional props use `?` and defaults when destructuring.
- Types are checked at build time; they do not ship to the browser.

On the Constellation site, templates in `src/layouts/templates/` define `Props` interfaces for `seoTitle`, `canonicalUrl`, chapter lists, etc. Pages must pass the names each template expects.

---

## Children (default slot)

Content between opening and closing tags becomes **slot content**:

```astro
<!-- Page file -->
<ArticleShell title="Guide">
  <h2>Section one</h2>
  <p>Body copy here.</p>
</ArticleShell>
```

```astro
<!-- ArticleShell.astro -->
---
const { title } = Astro.props;
---
<article>
  <h1>{title}</h1>
  <div class="body">
    <slot />
  </article>
</article>
```

Everything inside `<ArticleShell>...</ArticleShell>` in the parent renders where `<slot />` appears in the child.

This is how blog posts on our site work: the page file holds HTML; `BlogPostTemplate` wraps it in hero, sidebar, and footer chrome.

---

## `Astro` global (common APIs)

Available in frontmatter and sometimes referenced in docs:

| Property / method | Typical use |
|-------------------|-------------|
| `Astro.props` | Incoming attributes |
| `Astro.url` | URL of the current page (path, origin) |
| `Astro.generator` | Site generator meta string |
| `Astro.slots` | Check if a named slot has content |

Example: default canonical URL when a page omits one:

```astro
---
const { canonicalUrl = Astro.url.href } = Astro.props;
---
<link rel="canonical" href={canonicalUrl} />
```

---

## Component composition patterns

### Wrapper (layout) component

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="About">
  <main>...</main>
</BaseLayout>
```

### Presentational component

```astro
---
// TrustBadge.astro
interface Props { label: string; }
const { label } = Astro.props;
---
<span class="badge">{label}</span>
```

### List rendering in parent

```astro
---
const items = ['SEO', 'PPC', 'Web'];
---
<ul>
  {items.map(item => <li>{item}</li>)}
</ul>
```

---

## What components cannot do (by default)

- **No shared state** between components like React Context (unless you add a client framework island).
- **No re-render** when a prop “changes” in the browser—static pages are fixed at build time.
- **No hooks** (`useState`, etc.) in `.astro` frontmatter.

For interactive UI, add a `<script>` in the component, or a framework island with `client:*` directives (not used on Constellation’s site today).

---

## File naming conventions

| Convention | Example |
|------------|---------|
| PascalCase for components | `SiteHeader.astro`, `BlogPostTemplate.astro` |
| kebab-case or descriptive paths for pages | `law-firm-seo.astro` under `src/pages/blog/` |
| One default component per file | File name ≈ component name |

---

## Debugging props issues

| Symptom | Likely cause |
|---------|----------------|
| `undefined` in template | Prop name mismatch (e.g. `seoTitle` vs `title`) |
| Type error on build | Missing required prop or wrong type |
| Child content missing | Forgot `<slot />` in wrapper |
| Import error | Wrong relative path after file move |

On this repo, always trace: **page props → template → `BaseLayout` props**. See [ASTRO.md](../ASTRO.md#passing-seo-from-page-to-layout).

---

## Next

[4. Slots and layouts →](./04-slots-and-layouts.md)
