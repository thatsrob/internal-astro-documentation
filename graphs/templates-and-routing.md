# Templates and Routing

Template picker and relationships between page types.

## Template picker (decision flow)

```mermaid
flowchart TD
    START["New page in src/pages/"]
    START --> Q1{What are you building?}

    Q1 -->|Homepage| HP["HomepageTemplate<br/>(v2/v3 for experiments)"]
    Q1 -->|Blog article| BP["BlogPostTemplate<br/>+ HTML in default slot"]
    Q1 -->|City / geo landing| CT["CityTemplate<br/>~138 city pages"]
    Q1 -->|Practice area guide| PA["PracticeAreaTemplate"]
    Q1 -->|Service line page| SV["ServiceTemplate"]
    Q1 -->|Podcast episode| PC["PodcastTemplate<br/>YouTube embed"]
    Q1 -->|Case study| CS["CaseStudyNarrativeTemplate"]
    Q1 -->|Pillar guide + chapters| GD["GuidesTemplate"]
    Q1 -->|Agency review| AR["AgencyReviewTemplate"]
    Q1 -->|Book a call landing| BC["BookACallTemplate<br/>or custom book-call.astro"]
    Q1 -->|About, FAQ, privacy, blog index| CR["CoreTemplate"]
    Q1 -->|Unique one-off| DIRECT["BaseLayout + Header/Footer<br/>no template"]

    HP & BP & CT & PA & SV & PC & CS & GD & AR & BC & CR --> STACK["All render through<br/>BaseLayout + SiteHeader + SiteFooter"]
    DIRECT --> STACK
```

## Template catalog

```mermaid
mindmap
  root((templates/))
    Marketing
      HomepageTemplate
      HomepageTemplate v2 v3
      ServiceTemplate
      CoreTemplate
      BookACallTemplate
    Content
      BlogPostTemplate
      PracticeAreaTemplate
      GuidesTemplate
      AgencyReviewTemplate
    Geo & proof
      CityTemplate
      CaseStudyNarrativeTemplate
      PodcastTemplate
```

## Import path depth

```mermaid
flowchart TB
    subgraph DEPTH1["src/pages/foo.astro"]
        I1["../layouts/templates/X.astro"]
    end
    subgraph DEPTH2["src/pages/blog/foo.astro<br/>src/pages/examples/foo.astro"]
        I2["../../layouts/templates/X.astro"]
    end
    subgraph DEPTH3["src/pages/case-study/foo.astro"]
        I3["../../layouts/templates/X.astro"]
    end
```

## Example routes for QA

```mermaid
flowchart LR
    subgraph PROD["Production nav"]
        HOME["/"]
        BLOG["/blog/..."]
        CITIES["/attorney-marketing-*"]
        BOOK["/book-call/"]
    end

    subgraph EXAMPLES["/examples/ (dev & QA only)"]
        E1["homepage-example"]
        E2["blog-post-example"]
        E3["city-example"]
        E4["...one per template"]
    end

    EXAMPLES -.->|"mirror template props"| TEMPLATES["src/layouts/templates/"]
```

## Known template → BaseLayout bug

```mermaid
flowchart LR
    CR["CoreTemplate"]
    CR -->|"seoTitle={...} ❌"| BL["BaseLayout expects title"]
    BL --> BROKEN["Broken &lt;title&gt; on<br/>about-us, FAQ, privacy, blog index"]

    OTHER["Other templates"]
    OTHER -->|"title={seoTitle} ✓"| BL2["BaseLayout"]
```
