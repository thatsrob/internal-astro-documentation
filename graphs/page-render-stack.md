# Page Render Stack

How a visitor-facing page is assembled at build time.

## Default page composition

```mermaid
flowchart TB
    PAGE["src/pages/your-page.astro<br/>SEO props · template import · slot content"]
    TEMPLATE["src/layouts/templates/*.astro<br/>page-type chrome · sections · scoped styles"]
    BASE["BaseLayout.astro<br/>DOCTYPE · head · meta · global CSS"]
    HEADER["SiteHeader.astro<br/>nav · dropdowns · mobile menu"]
    CONTENT["[content / default slot]<br/>HTML body or template markup"]
    FOOTER["SiteFooter.astro<br/>links · social · sitemap"]

    PAGE --> TEMPLATE
    TEMPLATE --> BASE
    TEMPLATE --> HEADER
    TEMPLATE --> CONTENT
    TEMPLATE --> FOOTER
    BASE --> HEAD["&lt;head&gt;<br/>title · canonical · OG · Twitter"]
    BASE --> BODY["&lt;body&gt;<br/>&lt;slot /&gt;"]
```

## SEO prop flow

```mermaid
flowchart LR
    P["Page: seoTitle<br/>seoDescription<br/>canonicalUrl"]
    T["Template: maps props"]
    B["BaseLayout: title<br/>description<br/>canonicalUrl"]
    HTML["&lt;title&gt; · meta · link rel=canonical"]

    P --> T
    T -->|"title={seoTitle}"| B
    T -->|"description={seoDescription}"| B
    T -->|"canonicalUrl"| B
    B --> HTML
```

## Blog post with slot content

```mermaid
sequenceDiagram
    participant Page as blog/post.astro
    participant Blog as BlogPostTemplate
    participant Base as BaseLayout
    participant Browser as dist HTML

    Page->>Blog: seoTitle, chapters, publishDate
    Page->>Blog: default slot (article HTML)
    Blog->>Base: title, description, canonicalUrl
    Blog->>Blog: render hero, TOC, article-body
  Note over Blog: &lt;style is:global slot="head"&gt;
    Blog->>Browser: static HTML at build time
```

## Hand-built pages (no template)

```mermaid
flowchart TB
    subgraph TEMPLATE_PATH["Most pages (~90%)"]
        P1["page.astro"] --> T1["Template"] --> BL1["BaseLayout + Header/Footer"]
    end

    subgraph DIRECT_PATH["High-value custom pages"]
        P2["book-call.astro<br/>law-firm-seo-services.astro<br/>become-a-partner.astro"]
        P2 --> BL2["BaseLayout + SiteHeader + SiteFooter<br/>inline markup + embeds"]
    end
```

## Astro build-time execution

```mermaid
flowchart LR
    FM["Frontmatter ---<br/>runs once per page"]
    TMPL["Template expressions<br/>.map() · conditionals"]
    RENDER["Render to HTML string"]
    DIST["dist/path/index.html"]

    FM --> TMPL --> RENDER --> DIST

    subgraph RUNTIME["Not used on this site"]
        SSR["SSR per request"]
        ISLANDS["client:load islands"]
        API["API routes"]
    end

    style RUNTIME fill:#f9f9f9,stroke:#ccc
```
