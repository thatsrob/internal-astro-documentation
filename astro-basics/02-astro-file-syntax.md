# 2. The `.astro` file

Every Astro page and component is a single file with the extension **`.astro`**. Understanding its four regions is the foundation for everything else.

**Back to series:** [Astro basics index](./README.md) · **Previous:** [Introduction](./01-introduction.md)

---

## Anatomy of one file

```astro
---
// 1. FRONTMATTER (required fence, optional code)
import Card from './Card.astro';
const title = 'Hello';
---

<!-- 2. TEMPLATE (HTML + expressions) -->
<h1>{title}</h1>
<Card label="Learn more" />

<!-- 3. STYLE (optional, usually scoped) -->
<style>
  h1 { color: #1a3c34; }
</style>

<!-- 4. SCRIPT (optional, runs in the browser) -->
<script>
  console.log('This runs on the client after load');
</script>
```

| Section | Runs where | Purpose |
|---------|------------|---------|
| Frontmatter `---` | **Build time** (Node) | Imports, data, logic |
| Template | **Build time** → becomes HTML | What visitors see |
| `<style>` | Build time → CSS in output | Component styles |
| `<script>` | **Browser** (client) | DOM behavior, analytics hooks |

Most Constellation pages rely on **frontmatter + template** only. Client `<script>` is rare.

---

## Frontmatter

The frontmatter block is fenced with `---` and holds **JavaScript or TypeScript**:

```astro
---
const posts = [
  { title: 'Law firm SEO', slug: 'law-firm-seo' },
  { title: 'PPC for attorneys', slug: 'ppc-attorneys' },
];
---
```

Rules:

- The opening `---` must be the **first thing** in the file (only a UTF-8 BOM before it, if any).
- Runs **once per page** when Astro builds (or on each SSR request if configured).
- Can use `import`, `await` (top-level in async setups), `fetch`, file reads, etc.
- TypeScript works here when the project uses Astro’s TS setup (as in our repo).
- Cannot access `window` or `document`—those exist only in the browser.

Use frontmatter for: formatting dates, filtering lists, reading config, passing props to child components.

> If you need `---` inside the visible page, put it in the template as text or an HTML comment—it is not frontmatter unless it is at the very top.

---

## Template

The template is **HTML-like markup** with JavaScript expressions in curly braces:

```astro
<p>Published: {new Date('2024-06-01').getFullYear()}</p>
```

You can:

- Import and use other `.astro` components (PascalCase tags).
- Use standard HTML elements.
- Nest components and pass attributes as props.
- Use self-closing tags for components with no children: `<SiteHeader />`.

```astro
<img src="/images/hero.webp" alt={heroAlt} loading="lazy" />
```

Attributes can be dynamic: `class={isActive ? 'active' : ''}`. Boolean attributes can use shorthand: `disabled` means `disabled={true}`.

### HTML in templates

You can mix normal HTML comments, `<!-- like this -->`, with components. Astro does not re-parse arbitrary HTML inside slots as components—it outputs it as markup. That is why migrated WordPress HTML can live inside `<BlogPostTemplate>...</BlogPostTemplate>` unchanged.

### Only one frontmatter block

Each `.astro` file has exactly one `---` fence pair at the top. Put all imports and logic there, not in the template.

---

## `<style>` — component CSS

Styles in a `.astro` file are **scoped by default**: selectors only apply to elements in that component’s template.

```astro
<style>
  .card { padding: 1rem; }  /* becomes something like .card[data-astro-xxx] */
</style>
```

Common modifiers:

| Directive | Effect |
|-----------|--------|
| `is:global` | Selector applies site-wide (use sparingly) |
| `define:vars={{ color }}` | Pass frontmatter values into CSS |

Example with a named head slot (used on our site for page backgrounds):

```astro
<style is:global slot="head">
  body { background: #efefef; }
</style>
```

---

## `<script>` — client JavaScript

Scripts without a `type` attribute are **processed and bundled** by Astro. They run in the browser after the page loads.

```astro
<script>
  const button = document.querySelector('#menu-toggle');
  button?.addEventListener('click', () => { /* ... */ });
</script>
```

- **Do not** put secrets in client scripts—they ship to visitors.
- Prefer zero client JS for content pages; use islands or vanilla script only when needed.

Constellation’s site mostly avoids client scripts in favor of static HTML and vendor embeds (iframes, external widgets).

### Script directives (when you do need client JS)

| Directive | Behavior |
|-----------|----------|
| (default) | Bundled module, runs once when loaded |
| `is:inline` | Copied into HTML verbatim; no bundling |
| `define:vars={{ x }}` | Pass build-time values into the script |

```astro
<script is:inline>
  /* Exact source appears in the page — good for tiny one-offs */
</script>
```

Prefer a single small script per page over sprinkling many, so behavior stays easy to audit.

---

## Pages vs components

The **same file syntax** is used for both:

| Location | Role |
|----------|------|
| `src/pages/*.astro` | Becomes a **route** (URL) |
| Any other `src/**/*.astro` | **Component** only (no URL unless imported by a page) |

There is no separate “page class”—only file placement under `src/pages/` makes a file a page.

---

## Minimal complete example

**`src/components/Greeting.astro`**

```astro
---
interface Props {
  name: string;
}
const { name } = Astro.props;
---
<p class="greeting">Hello, {name}!</p>

<style>
  .greeting { font-weight: 600; }
</style>
```

**`src/pages/hello.astro`**

```astro
---
import Greeting from '../components/Greeting.astro';
---
<html lang="en">
  <body>
    <Greeting name="Constellation" />
  </body>
</html>
```

Build output: static file at `/hello/` (or `/hello.html` depending on trailing slash config).

On Constellation routes, the page file usually **does not** include `<html>`—it imports a template that wraps content in `BaseLayout`. The example above is minimal; see [04-slots-and-layouts.md](./04-slots-and-layouts.md).

---

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Using `window` in frontmatter | Move logic to `<script>` or pass data as props |
| Forgetting `---` fences | Frontmatter must start and end with `---` on their own lines |
| Import path wrong after moving file | Count `../` segments from current file to target |
| Expecting frontmatter to run on every click | It runs at build time (static) or per request (SSR) |
| Putting `<html>` in both page and `BaseLayout` | Only the root layout should own `<html>` and `<head>` |
| Editing only in `dev` before a big PR | Run `npm run build && npm run preview` — production bundling can differ |

---

## Next

[3. Components and props →](./03-components-and-props.md)
