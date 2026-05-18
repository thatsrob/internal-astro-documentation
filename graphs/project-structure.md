# Project Structure

Repository layout for the Constellation Marketing Astro site (`con-staging`).

## Repository tree

```mermaid
flowchart TB
    subgraph ROOT["con-staging /"]
        CONFIG["astro.config.mjs<br/>static output, site URL, redirects"]
        PKG["package.json<br/>Astro 6, Tailwind 4"]
        GHA[".github/workflows/deploy.yml"]
    end

    subgraph SRC["src/"]
        PAGES["pages/<br/>~396 routes · file-based routing"]
        LAYOUTS["layouts/"]
        COMPONENTS["components/<br/>SiteHeader · SiteFooter"]
        STYLES["styles/<br/>global.css · divi-theme-options.css"]
    end

    subgraph LAYOUTS_DETAIL["layouts/"]
        BASE["BaseLayout.astro<br/>html, head, meta, CSS links"]
        TEMPLATES["templates/<br/>13 page-type shells"]
    end

    subgraph PUBLIC["public/ (copied verbatim)"]
        IMAGES["images/<br/>WebP, wp-uploads"]
        CSS["css/<br/>production-extracted.css"]
        FONTS["fonts/<br/>Poppins woff2"]
        ROBOTS["robots.txt"]
    end

    subgraph TOOLING["offline tooling (not in dist/)"]
        SCRIPTS["scripts/<br/>Python + Node migration"]
        BACKSTOP["backstop_data/<br/>visual regression"]
        FUNCTIONS["functions/<br/>staging basic auth"]
    end

    ROOT --> SRC
    ROOT --> PUBLIC
    ROOT --> TOOLING
    LAYOUTS --> LAYOUTS_DETAIL
    SRC --> LAYOUTS
```

## Page scale by route family

```mermaid
pie title Approximate page count (~396 total)
    "Blog posts" : 161
    "attorney-marketing-* cities" : 66
    "law-firm-marketing-* cities" : 72
    "Other (services, guides, legal, etc.)" : 97
```

## src/pages routing model

```mermaid
flowchart LR
    subgraph FILES["Filesystem under src/pages/"]
        IDX["index.astro"]
        BLOG["blog/law-firm-seo.astro"]
        CITY["attorney-marketing-austin.astro"]
        NESTED["law-firm-seo/benefit.astro"]
        EXAMPLES["examples/blog-post-example.astro"]
    end

    subgraph URLS["Public URLs"]
        U1["/"]
        U2["/blog/law-firm-seo/"]
        U3["/attorney-marketing-austin/"]
        U4["/law-firm-seo/benefit/"]
        U5["/examples/blog-post-example/"]
    end

    IDX --> U1
    BLOG --> U2
    CITY --> U3
    NESTED --> U4
    EXAMPLES --> U5
```

## public/ vs src/ processing

```mermaid
flowchart LR
  subgraph BUILD["npm run build"]
    VITE["Astro + Vite"]
    COPY["Copy public/ as-is"]
    OUT["dist/"]
  end

  SRC_IMPORT["src/ imports<br/>components, bundled CSS"]
  PUBLIC_STATIC["public/<br/>/images, /css, /fonts"]

  SRC_IMPORT --> VITE
  PUBLIC_STATIC --> COPY
  VITE --> OUT
  COPY --> OUT
```
