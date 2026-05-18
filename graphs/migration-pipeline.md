# Migration Pipeline

WordPress/Divi to Astro: extract, transform, verify.

## End-to-end migration

```mermaid
flowchart TB
    WP["WordPress + Divi<br/>goconstellation.com (production)"]
    
    subgraph EXTRACT["1. Extract"]
        SCRAPE["Python scrapers<br/>build_blog_posts.py · build_city_pages.py"]
        REFORE["Refore CDN export<br/>homepage methodology"]
        FIRE["Firecrawl JSON<br/>build_astro_post.py"]
    end

    subgraph TRANSFORM["2. Transform"]
        ASTRO_FILES["src/pages/*.astro"]
        TEMPLATES["Wrap in BlogPostTemplate,<br/>CityTemplate, etc."]
        IMAGES["public/images/<br/>often WebP"]
        CSS["production-extracted.css<br/>template inline styles"]
    end

    subgraph VERIFY["3. Verify"]
        BACKSTOP["BackstopJS<br/>screenshot diff vs production"]
        FIXLOOP["fix-loop.mjs<br/>Claude + Backstop optional"]
        MANUAL["Manual QA in dev/preview"]
    end

    subgraph SHIP["4. Ship"]
        GIT["Git PR → main"]
        STAGE["Cloudflare con-staging"]
        LAUNCH["DNS cutover<br/>LAUNCH-CHECKLIST.md"]
    end

    WP --> EXTRACT
    EXTRACT --> TRANSFORM
    TRANSFORM --> VERIFY
    VERIFY --> SHIP
```

## Blog post scraper pipeline

```mermaid
flowchart LR
    URL["Production URL"]
    FETCH["fetch_html()"]
    META["extract_meta()<br/>title, description"]
    DATE["extract_publish_date()"]
    BODY["extract_body_content()<br/>Divi .et_pb_post_content"]
    CLEAN["clean_html()<br/>strip scripts, shortcodes"]
    TOC["extract_h2_chapters()"]
    WRITE["write_blog_astro()<br/>BlogPostTemplate wrapper"]
    FILE["src/pages/blog/slug.astro"]

    URL --> FETCH --> META
    FETCH --> DATE
    FETCH --> BODY --> CLEAN --> TOC --> WRITE --> FILE
```

## City page scraper pipeline

```mermaid
flowchart LR
    CITY_URL["Production city URL"]
    SCRAPE["Scrape HTML + meta"]
    DL["Download images"]
    WEBP["Convert to WebP<br/>Pillow"]
    PROPS["CityTemplate props<br/>city, state, heroImage"]
    OUT["src/pages/attorney-marketing-*.astro"]

    CITY_URL --> SCRAPE --> PROPS
    SCRAPE --> DL --> WEBP --> OUT
    PROPS --> OUT
```

## Tool map

```mermaid
flowchart TB
    subgraph PYTHON["Python 3"]
        BBP["build_blog_posts.py<br/>bulk blog + nested"]
        BAP["build_astro_post.py<br/>single Firecrawl JSON"]
        BCP["build_city_pages.py<br/>geo pages + images"]
    end

    subgraph NODE["Node"]
        SS["split-sections.mjs<br/>HTML → section files"]
        FL["fix-loop.mjs<br/>Backstop + Anthropic API"]
    end

    subgraph CONFIG["Config / docs"]
        BS["backstop.config.cjs"]
        RF["REFORE-APPROACH.md"]
    end

    PYTHON --> PAGES["src/pages/"]
    NODE --> PAGES
    BS --> FL
```

## Script overwrite behavior

```mermaid
flowchart LR
    BLOG["build_blog_posts.py"] -->|overwrites| OW1["Existing .astro files"]
    CITY["build_city_pages.py"] -->|skips if exists| SKIP["{slug}.astro present"]
    ASTRO_POST["build_astro_post.py"] -->|overwrites| OW2["Output file"]
    FIX["fix-loop.mjs"] -->|overwrites| HP["HomepageTemplate.astro"]
```

## Content sources after migration

```mermaid
flowchart TB
    subgraph GIT["Source of truth: Git"]
        BLOG["blog/*.astro · HTML in file"]
        CITIES["attorney-marketing-*.astro"]
        HOME["HomepageTemplate.astro"]
        ASSETS["public/images/"]
    end

    subgraph NO_CMS["Not in repo"]
        CMS["Headless CMS"]
        DB["Database"]
        WP_ADMIN["wp-admin publishing"]
    end

    style NO_CMS fill:#f5f5f5,stroke:#ccc
```
