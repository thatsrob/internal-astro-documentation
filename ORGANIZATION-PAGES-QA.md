# Organization Pages — QA URLs

Local preview base: **http://localhost:4321** (run `npm run dev` first).

Production reference: **https://goconstellation.com**

All pages live at `src/pages/<slug>.astro` → `/<slug>/`.

---

## Conference / event pages (7)

Scraped from conferences SPA. Watch for unstyled Tailwind classes, broken internal links, and layout differences vs WordPress.

| Slug | Local QA URL | Production |
|------|--------------|------------|
| `aba-techshow-2024-legal-technology-innovation` | http://localhost:4321/aba-techshow-2024-legal-technology-innovation/ | https://goconstellation.com/aba-techshow-2024-legal-technology-innovation/ |
| `clio-cloud-conference-2024` | http://localhost:4321/clio-cloud-conference-2024/ | https://goconstellation.com/clio-cloud-conference-2024/ |
| `game-changers-summit-2024` | http://localhost:4321/game-changers-summit-2024/ | https://goconstellation.com/game-changers-summit-2024/ |
| `iltacon-legal-technology-conference-guide` | http://localhost:4321/iltacon-legal-technology-conference-guide/ | https://goconstellation.com/iltacon-legal-technology-conference-guide/ |
| `legal-marketing-association-annual-conference` | http://localhost:4321/legal-marketing-association-annual-conference/ | https://goconstellation.com/legal-marketing-association-annual-conference/ |
| `mtmp-conference-guide-mass-tort-attorneys` | http://localhost:4321/mtmp-conference-guide-mass-tort-attorneys/ | https://goconstellation.com/mtmp-conference-guide-mass-tort-attorneys/ |
| `tbi-med-legal-conference` | http://localhost:4321/tbi-med-legal-conference/ | https://goconstellation.com/tbi-med-legal-conference/ |

---

## Organization / association pages (13)

Scraped from WordPress (Divi). Watch for layout overlap, invisible `et_animated` content, testimonial grids, and hotlinked `wp-content` images.

| Slug | Progress | Local QA URL | Production |
|------|----------|--------------|------------|
| `aapda` | 90% | http://localhost:4321/aapda/ | https://goconstellation.com/aapda/ |
| `advanced-investigation-software-solutions-for-legal-professionals` | 0% | http://localhost:4321/advanced-investigation-software-solutions-for-legal-professionals/ | https://goconstellation.com/advanced-investigation-software-solutions-for-legal-professionals/ |
| `american-immigration-lawyers-association` | 0% | http://localhost:4321/american-immigration-lawyers-association/ | https://goconstellation.com/american-immigration-lawyers-association/ |
| `asopica` | 0% | http://localhost:4321/asopica/ | https://goconstellation.com/asopica/ |
| `dc-criminal-defense-bar` | 0% | http://localhost:4321/dc-criminal-defense-bar/ | https://goconstellation.com/dc-criminal-defense-bar/ |
| `institute-well-being-law` | 0% | http://localhost:4321/institute-well-being-law/ | https://goconstellation.com/institute-well-being-law/ |
| `lawyers-committee` | 0% | http://localhost:4321/lawyers-committee/ | https://goconstellation.com/lawyers-committee/ |
| `michigan-bar-organization` | 0% | http://localhost:4321/michigan-bar-organization/ | https://goconstellation.com/michigan-bar-organization/ |
| `richmond-bar-organization` | 0% | http://localhost:4321/richmond-bar-organization/ | https://goconstellation.com/richmond-bar-organization/ |

---

## Guides / article-style (4)

WordPress content; generally closer to standard blog layout.

| Slug | Local QA URL | Production |
|------|--------------|------------|
| `should-i-join-the-legal-marketing-association-lma` | http://localhost:4321/should-i-join-the-legal-marketing-association-lma/ | https://goconstellation.com/should-i-join-the-legal-marketing-association-lma/ |
| `aila-conference-2024-guide` | http://localhost:4321/aila-conference-2024-guide/ | https://goconstellation.com/aila-conference-2024-guide/ |
| `ceo-lawyer-summit-2024` | http://localhost:4321/ceo-lawyer-summit-2024/ | https://goconstellation.com/ceo-lawyer-summit-2024/ |
| `justice-annual-convention-guide` | http://localhost:4321/justice-annual-convention-guide/ | https://goconstellation.com/justice-annual-convention-guide/ |

---

## Copy-paste checklist (local only)

```
http://localhost:4321/aba-techshow-2024-legal-technology-innovation/
http://localhost:4321/clio-cloud-conference-2024/
http://localhost:4321/game-changers-summit-2024/
http://localhost:4321/iltacon-legal-technology-conference-guide/
http://localhost:4321/legal-marketing-association-annual-conference/
http://localhost:4321/mtmp-conference-guide-mass-tort-attorneys/
http://localhost:4321/tbi-med-legal-conference/
http://localhost:4321/aapda/
http://localhost:4321/advanced-investigation-software-solutions-for-legal-professionals/
http://localhost:4321/american-immigration-lawyers-association/
http://localhost:4321/asopica/
http://localhost:4321/dc-criminal-defense-bar/
http://localhost:4321/institute-well-being-law/
http://localhost:4321/lawyers-committee/
http://localhost:4321/michigan-bar-organization/
http://localhost:4321/richmond-bar-organization/
http://localhost:4321/should-i-join-the-legal-marketing-association-lma/
http://localhost:4321/aila-conference-2024-guide/
http://localhost:4321/ceo-lawyer-summit-2024/
http://localhost:4321/justice-annual-convention-guide/
```

---

## Notes

- **Duplicate route:** `should-i-join-the-legal-marketing-association-lma` also exists at http://localhost:4321/blog/should-i-join-the-legal-marketing-association-lma/ — confirm canonical URL before launch.
- **AAPDA** uses `src/styles/org-aapda.css` + production hero HTML in `scripts/aapda-assets/hero-sections.html`; `hideBlogHero` + `hideSiteHeader` on `BlogPostTemplate` (this page only). Other Divi landings may need similar treatment.
- **Advanced Investigation** uses Theme Builder layout (`section_0_tb_body` hero + `section_1_tb_body` content), not the author-header Divi pattern. Scoped CSS: `src/styles/org-advanced-investigation.css`; `hideBlogHero` + `hideSidebar` on `BlogPostTemplate`. Nav overrides in that CSS only (full-width navy bar + gradient Book a Call CTA, not the green pill header).
- **Migration script:** `scripts/build_organization_pages.py`
- **Related docs:** [QA-PROCEDURES.md](./QA-PROCEDURES.md), [INDIVIDUAL-PAGE-MIGRATION.md](./INDIVIDUAL-PAGE-MIGRATION.md)
