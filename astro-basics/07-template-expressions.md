# 7. Template expressions

The template section of an `.astro` file is HTML with embedded JavaScript **expressions** in `{curly braces}`. This page covers the syntax you will use daily.

**Back to series:** [Astro basics index](./README.md) · **Previous:** [Build time vs runtime](./06-build-time-vs-runtime.md)

---

## Interpolation

Print any JavaScript value:

```astro
<p>{title}</p>
<p>{2 + 2}</p>
<p>{user.name}</p>
```

Values are escaped for HTML by default (safe against basic XSS from strings).

`undefined` and `null` render as nothing—useful for optional fields. `0` renders as `0` (unlike React, where `0` can be tricky in `{count && ...}` patterns).

---

## Attributes

Dynamic attributes use the same braces:

```astro
<img src={imageUrl} alt={altText} />
<a href={post.url}>{post.title}</a>
```

Boolean attributes:

```astro
<input disabled={isDisabled} />
<!-- or -->
<input disabled />
```

---

## Conditionals

### Inline logical AND

```astro
{showBanner && <div class="banner">Limited offer</div>}
```

Renders the element only when `showBanner` is truthy.

### Ternary

```astro
<p>{isMember ? 'Welcome back' : 'Sign up today'}</p>
```

### If / else blocks

For larger branches, use frontmatter or fragment wrappers:

```astro
---
const variant = 'dark';
---
{variant === 'dark' ? (
  <section class="hero hero--dark">...</section>
) : (
  <section class="hero hero--light">...</section>
)}
```

---

## Lists and `.map()`

Render arrays by mapping to elements:

```astro
---
const cities = ['Austin', 'Dallas', 'Houston'];
---
<ul>
  {cities.map(city => <li>{city}</li>)}
</ul>
```

With objects:

```astro
---
const links = [
  { href: '/seo/', label: 'SEO' },
  { href: '/ppc/', label: 'PPC' },
];
---
<nav>
  {links.map(link => (
    <a href={link.href}>{link.label}</a>
  ))}
</nav>
```

**Keys:** For static build-time lists, React-style `key` props are optional. Use a unique `key` if you later hydrate a list with a framework island.

---

## `class` and `class:list`

### String `class`

```astro
<div class={`card ${isFeatured ? 'card--featured' : ''}`}>
```

### `class:list` helper

```astro
<div class:list={['card', { 'card--featured': isFeatured }]}>
```

Prefer `class:list` when toggling multiple conditional classes.

### Spreading attributes

```astro
---
const linkProps = { href: '/book-call/', class: 'btn btn-primary' };
---
<a {...linkProps}>Book a call</a>
```

Works for any attribute object built in frontmatter.

---

## Astro template directives (quick reference)

Beyond `{expressions}`, Astro adds **compiler directives** on tags:

| Directive | Example | Purpose |
|-----------|---------|---------|
| `set:html` | `<div set:html={html} />` | Insert trusted HTML string |
| `set:text` | `<p set:text={msg} />` | Insert plain text node |
| `is:global` | `<style is:global>` | Unscoped CSS |
| `define:vars` | `<style define:vars={{ c }}>` | CSS variables from JS |
| `slot="head"` | `<style slot="head">` | Target named layout slot |
| `client:*` | `<Widget client:visible />` | Hydrate framework component (not used here) |

---

## `set:html` (raw HTML)

Inject HTML from a string **without escaping**:

```astro
<div set:html={trustedHtmlString} />
```

**Only use with trusted content** (your own migration output, sanitized CMS). Never use with raw user input.

Some Constellation pages use `set:html` for inline SVG icon strings from props.

---

## Fragments

Wrap multiple siblings without an extra DOM node:

```astro
<>
  <h2>Title</h2>
  <p>Paragraph</p>
</>
```

Or explicit `<Fragment>` in some versions; `<>` is common.

---

## Comments

HTML comments in templates:

```astro
<!-- This comment appears in output HTML -->
```

JavaScript comments belong in frontmatter, not inside `{expressions}` in the template.

---

## What you cannot do in templates

| Not allowed | Do instead |
|-------------|------------|
| `const x = 1` as a statement | Move to frontmatter |
| `if () { }` statements | Ternary, `&&`, or frontmatter logic |
| `await` in template body | `await` in frontmatter, assign to variable |

```astro
---
const formatted = await fetchSomething();
---
<p>{formatted}</p>
```

---

## HTML vs expression boundaries

```astro
<!-- Good -->
<p class="intro">{summary}</p>

<!-- Wrong: missing braces -->
<p class="intro">summary</p>

<!-- Good: attribute -->
<div class={active ? 'on' : 'off'} />
```

---

## Common patterns on marketing sites

### Format a date (frontmatter)

```astro
---
const { publishDate } = Astro.props;
const formatted = publishDate
  ? new Date(publishDate).toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
    })
  : null;
---
{formatted && <time datetime={publishDate}>{formatted}</time>}
```

### Repeat trust badges / features

```astro
{features.map(f => (
  <div class="feature">
    <h3>{f.title}</h3>
    <p>{f.body}</p>
  </div>
))}
```

### Optional SEO fields

```astro
{description && <meta name="description" content={description} />}
```

### Beware `{count && <Widget />}` when count can be 0

If `count` is `0`, the expression renders `0` in the DOM. Prefer `count > 0 && ...` or a ternary.

---

## Quick reference

| Task | Syntax |
|------|--------|
| Print value | `{value}` |
| Attribute | `href={url}` |
| Conditional block | `{cond && <Tag />}` |
| Either/or text | `{a ? b : c}` |
| List | `{items.map(i => <li>{i}</li>)}` |
| Classes | `class:list={[...]}` |
| Raw HTML | `set:html={str}` |

---

## Next

[8. Assets, images, and styles →](./08-assets-images-and-styles.md)
