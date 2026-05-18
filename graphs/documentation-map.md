# Documentation Map

How the internal docs relate to each other and to the codebase.

## Recommended reading order

```mermaid
flowchart TD
    START["New developer"]
    START --> CO["COMPREHENSIVE-OVERVIEW.md<br/>Start here"]
    CO --> INV["PAGE-INVENTORY.md<br/>Tracker templates & pages"]
    CO --> TODO["TODO.md<br/>Post-migration checklist"]
    CO --> ASTRO["ASTRO.md<br/>Framework on this site"]
    CO --> DEV["DEVELOPMENT.md<br/>Commands & workflow"]
    
    ASTRO --> TEMPLATES["TEMPLATES.md"]
    DEV --> CONTENT["CONTENT-GUIDE.md"]
    
    TEMPLATES --> STYLING["STYLING.md"]
    CONTENT --> SEO["SEO.md"]
    
    DEV --> DEPLOY["DEPLOYMENT.md"]
    SEO --> LAUNCH["LAUNCH-CHECKLIST.md"]
    
    DEV --> MIGRATE["MIGRATION-TOOLING.md"]
    MIGRATE --> VISUAL["VISUAL-QA.md"]
    
    DEPLOY --> INTEG["INTEGRATIONS.md"]
    
    CO --> SITE["SITE-OVERVIEW.md<br/>Shorter architecture reference"]
```

## Doc topics → codebase areas

```mermaid
flowchart LR
    subgraph DOCS["documents/"]
        D1["ASTRO.md"]
        D2["TEMPLATES.md"]
        D3["STYLING.md"]
        D4["SEO.md"]
        D5["DEPLOYMENT.md"]
        D6["MIGRATION-TOOLING.md"]
        D7["INTEGRATIONS.md"]
        D8["LAUNCH-CHECKLIST.md"]
    end

    subgraph CODE["Key code paths"]
        C1["astro.config.mjs"]
        C2["src/layouts/templates/"]
        C3["src/styles/ · public/css/"]
        C4["canonicalUrl in pages"]
        C5[".github/workflows/deploy.yml"]
        C6["scripts/"]
        C7["book-call · support-center"]
        C8["robots.txt · DNS cutover"]
    end

    D1 --> C1
    D2 --> C2
    D3 --> C3
    D4 --> C4
    D5 --> C5
    D6 --> C6
    D7 --> C7
    D8 --> C8
```

## Debugging decision tree

```mermaid
flowchart TD
    BUG["Something is wrong"]
    BUG --> Q1{Build fails?}
    Q1 -->|Yes| B1["Syntax in .astro<br/>invalid HTML in blog body<br/>wrong import path depth"]
    Q1 -->|No| Q2{404?}
    Q2 -->|Yes| B2["Missing file under src/pages/"]
    Q2 -->|No| Q3{Wrong title/meta?}
    Q3 -->|Yes| B3["Page → template → BaseLayout props<br/>CoreTemplate title bug"]
    Q3 -->|No| Q4{Layout broken?}
    Q4 -->|Yes| B4["Template markup or Divi CSS specificity"]
    Q4 -->|No| Q5{Style not applying?}
    Q5 -->|Yes| B5["Wrong CSS layer<br/>Divi vs global.css vs inline"]
    Q5 -->|No| Q6{Works in dev not preview?}
    Q6 -->|Yes| B6["Run npm run build + preview"]
    Q6 -->|No| Q7{Embed blank?}
    Q7 -->|Yes| B7["Third-party domain allowlist<br/>not Astro"]
```

## Graphs in this folder

```mermaid
flowchart LR
    G1["project-structure.md<br/>repo tree · routing · public vs src"]
    G2["page-render-stack.md<br/>page → template → layout"]
    G3["templates-and-routing.md<br/>picker · import paths"]
    G4["build-and-deploy.md<br/>CI/CD · environments"]
    G5["migration-pipeline.md<br/>WordPress → Astro tools"]
    G6["styling-and-integrations.md<br/>CSS layers · embeds"]
    G7["documentation-map.md<br/>this file"]
```
