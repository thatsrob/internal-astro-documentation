# Astro basics (internal reference)

A short, framework-focused guide to **Astro** for Constellation Marketing staff and contractors. These pages explain how Astro works as a tool—not how our WordPress migration repo is organized.

**Use this series when you need to:**

- Understand what an `.astro` file is before touching `src/pages/`
- Learn vocabulary (frontmatter, slot, island, static output)
- Onboard someone who knows HTML/CSS but not Astro

**Use [ASTRO.md](../ASTRO.md) when you need to:**

- See how Astro is configured and used **on the Constellation site**
- Debug this repo’s page → template → layout stack
- Compare Astro to our legacy WordPress setup

---

## Reading order

| # | Document | What you will learn |
|---|----------|---------------------|
| 1 | [Introduction](./01-introduction.md) | What Astro is, what problems it solves, when to choose it |
| 2 | [The `.astro` file](./02-astro-file-syntax.md) | Frontmatter, template, `<script>`, `<style>` |
| 3 | [Components and props](./03-components-and-props.md) | Importing components, passing data, TypeScript props |
| 4 | [Slots and layouts](./04-slots-and-layouts.md) | Wrapping pages, default and named slots |
| 5 | [Routing and pages](./05-routing-and-pages.md) | File-based URLs, dynamic routes, `index` files |
| 6 | [Build time vs runtime](./06-build-time-vs-runtime.md) | When code runs, static vs server output |
| 7 | [Template expressions](./07-template-expressions.md) | `{expressions}`, conditionals, lists, `class` |

---

## Official Astro resources

- [docs.astro.build](https://docs.astro.build) — canonical reference
- [Getting started](https://docs.astro.build/en/getting-started/)
- [Tutorial: Build a blog](https://docs.astro.build/en/tutorial/0-introduction/)

---

## Related internal docs

| Topic | Document |
|-------|----------|
| Astro on the Constellation site | [ASTRO.md](../ASTRO.md) |
| Day-to-day commands | [DEVELOPMENT.md](../DEVELOPMENT.md) |
| Page templates in this repo | [TEMPLATES.md](../TEMPLATES.md) |
| New developer onboarding | [COMPREHENSIVE-OVERVIEW.md](../COMPREHENSIVE-OVERVIEW.md) |
| Full doc index | [graphs/documentation-map.md](../graphs/documentation-map.md) |
