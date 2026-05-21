# Blog QA cheat sheet

**Generated:** 2026-05-21 16:40 UTC · Regenerate: `.venv/bin/python scripts/generate_blog_qa_cheatsheet.py`

Side-by-side QA: open **legacy** (WordPress) and **local** (Astro) in two tabs. Use **staging** after deploy.

**Runbook:** [BLOG-QA.md](./BLOG-QA.md) · **Tracker:** [blogs-to-do.md](./blogs-to-do.md)

---

## Progress

| Track | Bar | Done |
|-------|-----|------:|
| **Migration** (sitemap → `.astro`) | ████████████████████████ **100%** (185/185) | 185/185 |
| **Body copy QA** (full-body scrape) | █░░░░░░░░░░░░░░░░░░░░░░░ **5%** (3/55) | 3/55 |
| **Layout QA** (case studies) | ██░░░░░░░░░░░░░░░░░░░░░░ **8%** (1/13) | 1/13 |
| **Meta QA** (title/desc/canonical) | ░░░░░░░░░░░░░░░░░░░░░░░░ **0%** (0/55) | 0/55 |

**Blog files in repo:** `216` under `src/pages/blog/` · **Full-body log:** `55` slugs · **Est. stale scrape:** ~161

**Legend:** ✅ done · ⬜ pending · 📋 case study · ⚠️ verify title vs legacy

---

## URL patterns

| Environment | Pattern |
|-------------|---------|
| **Local** | `http://localhost:4321/blog/{slug}/` |
| **Legacy (production WP)** | `https://goconstellation.com/{slug}/` |
| **Staging (Astro)** | `https://staging.goconstellation.com/blog/{slug}/` |

Canonical in frontmatter uses **legacy** root path. Launch redirects may map `/slug/` → `/blog/slug/`.

---

## Per-post checklist

| Slug | Local | Legacy | Staging | Body | Layout | Meta | Notes |
|------|-------|--------|---------|:----:|:------:|:----:|-------|
| `attorney-directories` | [local](http://localhost:4321/blog/attorney-directories/) | [legacy](https://goconstellation.com/attorney-directories/) | [staging](https://staging.goconstellation.com/blog/attorney-directories/) | ✅ | ⬜ | ⬜ |   |
| `avoid-internet-scams` | [local](http://localhost:4321/blog/avoid-internet-scams/) | [legacy](https://goconstellation.com/avoid-internet-scams/) | [staging](https://staging.goconstellation.com/blog/avoid-internet-scams/) | ✅ | ⬜ | ⬜ |   |
| `best-law-blogs` | [local](http://localhost:4321/blog/best-law-blogs/) | [legacy](https://goconstellation.com/best-law-blogs/) | [staging](https://staging.goconstellation.com/blog/best-law-blogs/) | ✅ | ⬜ | ⬜ |   |
| *(case studies)* | | | | | | | |
| `ai-chatbot-case-study-mullen-law-firm` | [local](http://localhost:4321/blog/ai-chatbot-case-study-mullen-law-firm/) | [legacy](https://goconstellation.com/ai-chatbot-case-study-mullen-law-firm/) | [staging](https://staging.goconstellation.com/blog/ai-chatbot-case-study-mullen-law-firm/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `ai-chatbot-case-study-philip-kim-law` | [local](http://localhost:4321/blog/ai-chatbot-case-study-philip-kim-law/) | [legacy](https://goconstellation.com/ai-chatbot-case-study-philip-kim-law/) | [staging](https://staging.goconstellation.com/blog/ai-chatbot-case-study-philip-kim-law/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `bilingual-seo-breakthrough-case-study` | [local](http://localhost:4321/blog/bilingual-seo-breakthrough-case-study/) | [legacy](https://goconstellation.com/bilingual-seo-breakthrough-case-study/) | [staging](https://staging.goconstellation.com/blog/bilingual-seo-breakthrough-case-study/) | ⬜ | ✅ | ⬜ | 📋 CS  |
| `elder-law-webinar-case-study` | [local](http://localhost:4321/blog/elder-law-webinar-case-study/) | [legacy](https://goconstellation.com/elder-law-webinar-case-study/) | [staging](https://staging.goconstellation.com/blog/elder-law-webinar-case-study/) | ⬜ | ✅ | ⬜ | 📋 CS  |
| `high-value-family-law-cases-case-study` | [local](http://localhost:4321/blog/high-value-family-law-cases-case-study/) | [legacy](https://goconstellation.com/high-value-family-law-cases-case-study/) | [staging](https://staging.goconstellation.com/blog/high-value-family-law-cases-case-study/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `law-office-of-valery-nechay` | [local](http://localhost:4321/blog/law-office-of-valery-nechay/) | [legacy](https://goconstellation.com/law-office-of-valery-nechay/) | [staging](https://staging.goconstellation.com/blog/law-office-of-valery-nechay/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `law-offices-cheryl-davis` | [local](http://localhost:4321/blog/law-offices-cheryl-davis/) | [legacy](https://goconstellation.com/law-offices-cheryl-davis/) | [staging](https://staging.goconstellation.com/blog/law-offices-cheryl-davis/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `missouri-dwi-criminal-law-center` | [local](http://localhost:4321/blog/missouri-dwi-criminal-law-center/) | [legacy](https://goconstellation.com/missouri-dwi-criminal-law-center/) | [staging](https://staging.goconstellation.com/blog/missouri-dwi-criminal-law-center/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `paul-black-office` | [local](http://localhost:4321/blog/paul-black-office/) | [legacy](https://goconstellation.com/paul-black-office/) | [staging](https://staging.goconstellation.com/blog/paul-black-office/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `philip-kim-law` | [local](http://localhost:4321/blog/philip-kim-law/) | [legacy](https://goconstellation.com/philip-kim-law/) | [staging](https://staging.goconstellation.com/blog/philip-kim-law/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `seo-case-study-hirsch-law-group` | [local](http://localhost:4321/blog/seo-case-study-hirsch-law-group/) | [legacy](https://goconstellation.com/seo-case-study-hirsch-law-group/) | [staging](https://staging.goconstellation.com/blog/seo-case-study-hirsch-law-group/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `seo-growth-case-study-sabbeth-law` | [local](http://localhost:4321/blog/seo-growth-case-study-sabbeth-law/) | [legacy](https://goconstellation.com/seo-growth-case-study-sabbeth-law/) | [staging](https://staging.goconstellation.com/blog/seo-growth-case-study-sabbeth-law/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| `tadeo-silva-law-immigration-attorneys` | [local](http://localhost:4321/blog/tadeo-silva-law-immigration-attorneys/) | [legacy](https://goconstellation.com/tadeo-silva-law-immigration-attorneys/) | [staging](https://staging.goconstellation.com/blog/tadeo-silva-law-immigration-attorneys/) | ⬜ | ⬜ | ⬜ | 📋 CS  |
| *(verify title)* | | | | | | | |
| `aila-conference` | [local](http://localhost:4321/blog/aila-conference/) | [legacy](https://goconstellation.com/aila-conference/) | [staging](https://staging.goconstellation.com/blog/aila-conference/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| `game-changers-summit` | [local](http://localhost:4321/blog/game-changers-summit/) | [legacy](https://goconstellation.com/game-changers-summit/) | [staging](https://staging.goconstellation.com/blog/game-changers-summit/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| `iltacon` | [local](http://localhost:4321/blog/iltacon/) | [legacy](https://goconstellation.com/iltacon/) | [staging](https://staging.goconstellation.com/blog/iltacon/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| `missouri-bar-conference` | [local](http://localhost:4321/blog/missouri-bar-conference/) | [legacy](https://goconstellation.com/missouri-bar-conference/) | [staging](https://staging.goconstellation.com/blog/missouri-bar-conference/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| `solo-small-firm-conference` | [local](http://localhost:4321/blog/solo-small-firm-conference/) | [legacy](https://goconstellation.com/solo-small-firm-conference/) | [staging](https://staging.goconstellation.com/blog/solo-small-firm-conference/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| `tools-from-aba-techshow` | [local](http://localhost:4321/blog/tools-from-aba-techshow/) | [legacy](https://goconstellation.com/tools-from-aba-techshow/) | [staging](https://staging.goconstellation.com/blog/tools-from-aba-techshow/) | ⬜ | ⬜ | ⬜ |  ⚠️ title |
| *(all other full-body scrapes)* | | | | | | | |
| `attorneypages-com-lawyer-directory` | [local](http://localhost:4321/blog/attorneypages-com-lawyer-directory/) | [legacy](https://goconstellation.com/attorneypages-com-lawyer-directory/) | [staging](https://staging.goconstellation.com/blog/attorneypages-com-lawyer-directory/) | ⬜ | ⬜ | ⬜ |   |
| `automation-large-law-firm` | [local](http://localhost:4321/blog/automation-large-law-firm/) | [legacy](https://goconstellation.com/automation-large-law-firm/) | [staging](https://staging.goconstellation.com/blog/automation-large-law-firm/) | ⬜ | ⬜ | ⬜ |   |
| `boost-your-law-firms-growth-with-benchmarking` | [local](http://localhost:4321/blog/boost-your-law-firms-growth-with-benchmarking/) | [legacy](https://goconstellation.com/boost-your-law-firms-growth-with-benchmarking/) | [staging](https://staging.goconstellation.com/blog/boost-your-law-firms-growth-with-benchmarking/) | ⬜ | ⬜ | ⬜ |   |
| `boosting-conversions-with-a-great-intake-system` | [local](http://localhost:4321/blog/boosting-conversions-with-a-great-intake-system/) | [legacy](https://goconstellation.com/boosting-conversions-with-a-great-intake-system/) | [staging](https://staging.goconstellation.com/blog/boosting-conversions-with-a-great-intake-system/) | ⬜ | ⬜ | ⬜ |   |
| `create-law-firm-vision` | [local](http://localhost:4321/blog/create-law-firm-vision/) | [legacy](https://goconstellation.com/create-law-firm-vision/) | [staging](https://staging.goconstellation.com/blog/create-law-firm-vision/) | ⬜ | ⬜ | ⬜ |   |
| `dont-buy-an-expensive-website` | [local](http://localhost:4321/blog/dont-buy-an-expensive-website/) | [legacy](https://goconstellation.com/dont-buy-an-expensive-website/) | [staging](https://staging.goconstellation.com/blog/dont-buy-an-expensive-website/) | ⬜ | ⬜ | ⬜ |   |
| `elevating-your-practice-innovative-medical-malpractice-lawyer-marketing-strategies` | [local](http://localhost:4321/blog/elevating-your-practice-innovative-medical-malpractice-lawyer-marketing-strategies/) | [legacy](https://goconstellation.com/elevating-your-practice-innovative-medical-malpractice-lawyer-marketing-strategies/) | [staging](https://staging.goconstellation.com/blog/elevating-your-practice-innovative-medical-malpractice-lawyer-marketing-strategies/) | ⬜ | ⬜ | ⬜ |   |
| `enhancing-legal-practice-visibility-hispanic-marketing-strategies-for-attorneys` | [local](http://localhost:4321/blog/enhancing-legal-practice-visibility-hispanic-marketing-strategies-for-attorneys/) | [legacy](https://goconstellation.com/enhancing-legal-practice-visibility-hispanic-marketing-strategies-for-attorneys/) | [staging](https://staging.goconstellation.com/blog/enhancing-legal-practice-visibility-hispanic-marketing-strategies-for-attorneys/) | ⬜ | ⬜ | ⬜ |   |
| `how-much-legal-directories-cost` | [local](http://localhost:4321/blog/how-much-legal-directories-cost/) | [legacy](https://goconstellation.com/how-much-legal-directories-cost/) | [staging](https://staging.goconstellation.com/blog/how-much-legal-directories-cost/) | ⬜ | ⬜ | ⬜ |   |
| `how-to-get-a-5-star-review-from-a-client` | [local](http://localhost:4321/blog/how-to-get-a-5-star-review-from-a-client/) | [legacy](https://goconstellation.com/how-to-get-a-5-star-review-from-a-client/) | [staging](https://staging.goconstellation.com/blog/how-to-get-a-5-star-review-from-a-client/) | ⬜ | ⬜ | ⬜ |   |
| `how-to-manage-a-law-firm` | [local](http://localhost:4321/blog/how-to-manage-a-law-firm/) | [legacy](https://goconstellation.com/how-to-manage-a-law-firm/) | [staging](https://staging.goconstellation.com/blog/how-to-manage-a-law-firm/) | ⬜ | ⬜ | ⬜ |   |
| `how-to-minimize-your-tax-liabilities-as-a-self-employed-attorney` | [local](http://localhost:4321/blog/how-to-minimize-your-tax-liabilities-as-a-self-employed-attorney/) | [legacy](https://goconstellation.com/how-to-minimize-your-tax-liabilities-as-a-self-employed-attorney/) | [staging](https://staging.goconstellation.com/blog/how-to-minimize-your-tax-liabilities-as-a-self-employed-attorney/) | ⬜ | ⬜ | ⬜ |   |
| `how-to-not-rank-your-law-firm-on-google` | [local](http://localhost:4321/blog/how-to-not-rank-your-law-firm-on-google/) | [legacy](https://goconstellation.com/how-to-not-rank-your-law-firm-on-google/) | [staging](https://staging.goconstellation.com/blog/how-to-not-rank-your-law-firm-on-google/) | ⬜ | ⬜ | ⬜ |   |
| `instagram-lawyer-navigating-legal-waters-in-social-media` | [local](http://localhost:4321/blog/instagram-lawyer-navigating-legal-waters-in-social-media/) | [legacy](https://goconstellation.com/instagram-lawyer-navigating-legal-waters-in-social-media/) | [staging](https://staging.goconstellation.com/blog/instagram-lawyer-navigating-legal-waters-in-social-media/) | ⬜ | ⬜ | ⬜ |   |
| `justia-lawyers-guide-to-claiming-optimizing-profiles` | [local](http://localhost:4321/blog/justia-lawyers-guide-to-claiming-optimizing-profiles/) | [legacy](https://goconstellation.com/justia-lawyers-guide-to-claiming-optimizing-profiles/) | [staging](https://staging.goconstellation.com/blog/justia-lawyers-guide-to-claiming-optimizing-profiles/) | ⬜ | ⬜ | ⬜ |   |
| `law-firm-listing` | [local](http://localhost:4321/blog/law-firm-listing/) | [legacy](https://goconstellation.com/law-firm-listing/) | [staging](https://staging.goconstellation.com/blog/law-firm-listing/) | ⬜ | ⬜ | ⬜ |   |
| `law-firm-newsletter` | [local](http://localhost:4321/blog/law-firm-newsletter/) | [legacy](https://goconstellation.com/law-firm-newsletter/) | [staging](https://staging.goconstellation.com/blog/law-firm-newsletter/) | ⬜ | ⬜ | ⬜ |   |
| `lawyer-com-lawyer-directory` | [local](http://localhost:4321/blog/lawyer-com-lawyer-directory/) | [legacy](https://goconstellation.com/lawyer-com-lawyer-directory/) | [staging](https://staging.goconstellation.com/blog/lawyer-com-lawyer-directory/) | ⬜ | ⬜ | ⬜ |   |
| `lawyer-influencers-shaping-legal-trends-on-social-media` | [local](http://localhost:4321/blog/lawyer-influencers-shaping-legal-trends-on-social-media/) | [legacy](https://goconstellation.com/lawyer-influencers-shaping-legal-trends-on-social-media/) | [staging](https://staging.goconstellation.com/blog/lawyer-influencers-shaping-legal-trends-on-social-media/) | ⬜ | ⬜ | ⬜ |   |
| `lawyers-annual-content-optimization` | [local](http://localhost:4321/blog/lawyers-annual-content-optimization/) | [legacy](https://goconstellation.com/lawyers-annual-content-optimization/) | [staging](https://staging.goconstellation.com/blog/lawyers-annual-content-optimization/) | ⬜ | ⬜ | ⬜ |   |
| `lawyers-com-lawyer-directory` | [local](http://localhost:4321/blog/lawyers-com-lawyer-directory/) | [legacy](https://goconstellation.com/lawyers-com-lawyer-directory/) | [staging](https://staging.goconstellation.com/blog/lawyers-com-lawyer-directory/) | ⬜ | ⬜ | ⬜ |   |
| `legal-advertising-rules` | [local](http://localhost:4321/blog/legal-advertising-rules/) | [legacy](https://goconstellation.com/legal-advertising-rules/) | [staging](https://staging.goconstellation.com/blog/legal-advertising-rules/) | ⬜ | ⬜ | ⬜ |   |
| `legal-project-management-simplified` | [local](http://localhost:4321/blog/legal-project-management-simplified/) | [legacy](https://goconstellation.com/legal-project-management-simplified/) | [staging](https://staging.goconstellation.com/blog/legal-project-management-simplified/) | ⬜ | ⬜ | ⬜ |   |
| `legaladvice-com-lawyer-directory` | [local](http://localhost:4321/blog/legaladvice-com-lawyer-directory/) | [legacy](https://goconstellation.com/legaladvice-com-lawyer-directory/) | [staging](https://staging.goconstellation.com/blog/legaladvice-com-lawyer-directory/) | ⬜ | ⬜ | ⬜ |   |
| `marketing-attribution-law-firms` | [local](http://localhost:4321/blog/marketing-attribution-law-firms/) | [legacy](https://goconstellation.com/marketing-attribution-law-firms/) | [staging](https://staging.goconstellation.com/blog/marketing-attribution-law-firms/) | ⬜ | ⬜ | ⬜ |   |
| `mastering-marketing-attribution-for-law-firms` | [local](http://localhost:4321/blog/mastering-marketing-attribution-for-law-firms/) | [legacy](https://goconstellation.com/mastering-marketing-attribution-for-law-firms/) | [staging](https://staging.goconstellation.com/blog/mastering-marketing-attribution-for-law-firms/) | ⬜ | ⬜ | ⬜ |   |
| `multi-location-marketing-for-law-firms` | [local](http://localhost:4321/blog/multi-location-marketing-for-law-firms/) | [legacy](https://goconstellation.com/multi-location-marketing-for-law-firms/) | [staging](https://staging.goconstellation.com/blog/multi-location-marketing-for-law-firms/) | ⬜ | ⬜ | ⬜ |   |
| `social-media-content-planning-law-firms` | [local](http://localhost:4321/blog/social-media-content-planning-law-firms/) | [legacy](https://goconstellation.com/social-media-content-planning-law-firms/) | [staging](https://staging.goconstellation.com/blog/social-media-content-planning-law-firms/) | ⬜ | ⬜ | ⬜ |   |
| `the-money-mindset-with-margarita-eberline` | [local](http://localhost:4321/blog/the-money-mindset-with-margarita-eberline/) | [legacy](https://goconstellation.com/the-money-mindset-with-margarita-eberline/) | [staging](https://staging.goconstellation.com/blog/the-money-mindset-with-margarita-eberline/) | ⬜ | ⬜ | ⬜ |   |
| `tiktok-lawyer-unveils-legal-tips-for-content-creators` | [local](http://localhost:4321/blog/tiktok-lawyer-unveils-legal-tips-for-content-creators/) | [legacy](https://goconstellation.com/tiktok-lawyer-unveils-legal-tips-for-content-creators/) | [staging](https://staging.goconstellation.com/blog/tiktok-lawyer-unveils-legal-tips-for-content-creators/) | ⬜ | ⬜ | ⬜ |   |
| `using-video-to-grow-your-practice` | [local](http://localhost:4321/blog/using-video-to-grow-your-practice/) | [legacy](https://goconstellation.com/using-video-to-grow-your-practice/) | [staging](https://staging.goconstellation.com/blog/using-video-to-grow-your-practice/) | ⬜ | ⬜ | ⬜ |   |
| `workers-compensation-leads` | [local](http://localhost:4321/blog/workers-compensation-leads/) | [legacy](https://goconstellation.com/workers-compensation-leads/) | [staging](https://staging.goconstellation.com/blog/workers-compensation-leads/) | ⬜ | ⬜ | ⬜ |   |
| `youtube-for-lawyers` | [local](http://localhost:4321/blog/youtube-for-lawyers/) | [legacy](https://goconstellation.com/youtube-for-lawyers/) | [staging](https://staging.goconstellation.com/blog/youtube-for-lawyers/) | ⬜ | ⬜ | ⬜ |   |

---

## Redirect aliases (legacy sitemap only)

| Legacy sitemap slug | Redirects to | Local | Legacy |
|--------------------|--------------|-------|--------|
| `ways-in-which-artificial-intelligence-can-benefit-your-small-law-firm` | `/ai-for-lawyers/` | [local](http://localhost:4321/blog/ai-for-lawyers/) | [https://goconstellation.com/ways-in-which-artificial-intelligence-can-benefit-your-small-law-firm/](https://goconstellation.com/ways-in-which-artificial-intelligence-can-benefit-your-small-law-firm/) → 301 |

---

## Quick verification (copy/paste)

```
[ ] Intro paragraphs present (not starting at first H2 only)
[ ] Chapter list matches legacy H2 count
[ ] No Patrick CTA / GUIDE CHAPTERS / You might also like in body
[ ] Images load (note wp-content hotlinks)
[ ] Case study: no double hero / header overlap at 375px + 1440px
[ ] seoTitle matches legacy <title> (especially ⚠️ rows)
```

---

## Update progress

1. Mark **Body** ✅ in this file after verifying copy (edit `PILOTS_VERIFIED` in generator or add `QA_VERIFIED` in script).
2. Update **Body QA** column in [blogs-to-do.md](./blogs-to-do.md).
3. Re-run: `.venv/bin/python scripts/generate_blog_qa_cheatsheet.py`
