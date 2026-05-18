# Styling and Integrations

CSS layers and third-party services.

## Three CSS layers (every page)

```mermaid
flowchart TB
    PAGE["Every public page"]
    
    PAGE --> L1["Layer 1: Legacy Divi<br/>public/css/production-extracted.css<br/>~488KB · linked in BaseLayout"]
    PAGE --> L2["Layer 2: Astro global<br/>src/styles/global.css<br/>Tailwind 4 · Poppins · brand tokens"]
    PAGE --> L3["Layer 3: Template/page<br/>scoped &lt;style&gt; · inline style=&quot;&quot;<br/>slot=&quot;head&quot; overrides"]

    L2 --> DTO["divi-theme-options.css<br/>WP custom CSS snapshot"]

    subgraph DEBUG["If style 'does nothing'"]
        D1["Wrong layer?"]
        D2["Divi specificity wins?"]
        D3["Need is:global for slot HTML?"]
    end

    L1 & L2 & L3 -.-> DEBUG
```

## CSS load order in BaseLayout

```mermaid
sequenceDiagram
    participant BL as BaseLayout
    participant Vite as Bundled global.css
    participant Divi as production-extracted.css
    participant Head as slot name=head

    BL->>Vite: import global.css
    Note over Vite: Tailwind + fonts + @theme tokens
    BL->>Divi: link /css/production-extracted.css
    BL->>Head: per-page template styles
```

## Page-specific Divi CSS (reference only)

```mermaid
flowchart LR
    subgraph GLOBAL["Loaded on every page"]
        PE["production-extracted.css"]
    end

    subgraph REF["public/css/ — not auto-linked"]
        DH["divi-homepage.css"]
        DC["divi-city.css"]
        DB["divi-blog-post.css"]
        DP["divi-practice-area.css"]
        DS["divi-service.css"]
    end

    REF -.->|"migration artifacts"| TEMPLATES["Template inline styles"]
    GLOBAL --> BL["BaseLayout"]
```

## Visitor-facing integrations

```mermaid
flowchart TB
    subgraph STATIC["Most pages"]
        HTML["Static HTML only<br/>no third-party scripts"]
    end

    subgraph CONVERSION["Conversion"]
        BC["/book-call/<br/>LeadConnector / GoHighLevel<br/>iframe + form_embed.js"]
        SC["/support-center/<br/>ClickUp form iframe"]
    end

    subgraph MEDIA["Media"]
        YT["/podcasts/*<br/>YouTube iframe"]
        SOC["Footer / header<br/>outbound social links"]
    end

    subgraph GAP["Not in Astro yet"]
        GTM["Google Tag Manager<br/>GTM-PDGND8M on WordPress only"]
        NF["Ninja Forms in migrated HTML<br/>dead — use book-call / support-center"]
    end

    style GAP fill:#fff8e6,stroke:#d4a017
```

## Integration matrix

```mermaid
flowchart LR
    subgraph PAGES["Key routes"]
        P1["book-call.astro"]
        P2["support-center.astro"]
        P3["podcasts/*.astro"]
    end

    subgraph SERVICES["External services"]
        GHL["api.leadconnectorhq.com"]
        CU["forms.clickup.com"]
        YT["youtube.com embed"]
    end

    P1 --> GHL
    P2 --> CU
    P3 --> YT
```

## Dev-only tooling (not visitor-facing)

```mermaid
flowchart TB
    ANTH["ANTHROPIC_API_KEY"] --> FL["scripts/fix-loop.mjs"]
    FL --> HP["Patches HomepageTemplate.astro"]
    BS["BackstopJS"] --> FL
    BS --> REPORT["backstop_data/<br/>HTML diff reports"]
    REFORE["Refore export"] --> HOME["Homepage migration"]
```
