# Accessibility

What accessibility means for this migration, what is already in decent shape, what is known to be weak, and how to test before launch without pretending the site is fully WCAG-certified.

Legal marketing sites often mention ADA or accessibility in content; that does not automatically make the rebuilt site compliant. Treat accessibility as **risk reduction on conversion paths** first, then a broader audit if the business requires it.

---

## Scope for this project

### Launch gate (realistic)

Phase 4 QA treats accessibility as a **spot-check**, not a full audit:

- Keyboard can reach main navigation and primary CTAs
- Booking and support iframes are usable
- Privacy and terms links work from the footer
- No obvious blockers on `/`, `/book-call/`, and a sample of inner pages

**Time budget:** about 2–3 hours for a first pass (see QA procedures). That is enough to catch embarrassing issues, not enough to certify WCAG 2.2 AA across ~400 pages.

### Post-launch (optional)

- Full WCAG audit with a specialist or tool like axe Monitor
- Sitewide color-contrast pass after CSS diet
- Systematic fix of migrated blog HTML (headings, lists, alt text)
- Formal VPAT or client-facing accessibility statement if sales requires it

The master TODO lists a full WCAG audit as **P2 / fast-follow**—appropriate unless counsel or a client contract says otherwise.

---

## What is in good shape today

### Global header and footer

`SiteHeader.astro` includes several baseline patterns:

- Logo link has `aria-label="Constellation Marketing - Home"`
- Main nav has `aria-label="Main navigation"`
- Dropdown triggers use `aria-expanded` and `aria-haspopup`
- Decorative chevrons use `aria-hidden="true"`
- Mobile menu button has `aria-label="Open menu"` and toggles `aria-expanded`
- Footer social links use `aria-label` (Facebook, Instagram, LinkedIn, etc.)
- Footer logo image has alt text

`SiteFooter.astro` exposes legal links (`privacy-policy`, `terms-of-service`) that QA should confirm load.

### Some templates and pages

- `PracticeAreaTemplate` case-study carousel: previous/next buttons with `aria-label`
- `HomepageTemplate-v3` testimonial dots: `aria-label` per slide
- Some blog posts (e.g. marketing guide, law firm marketing ideas) use accordion buttons with `aria-expanded`

These are uneven—migrated Divi HTML on other posts may not follow the same pattern.

---

## Known gaps and risks

### Mobile navigation (high priority)

The header toggles a `mobile-open` class on the document body when the hamburger is pressed, but **CSS to show the mobile menu panel is missing or incomplete** on small viewports. Keyboard and screen-reader users on phones may not get a working menu.

**Test:** Resize below 1024px, open menu, tab through links. If the menu does not appear or focus is trapped incorrectly, treat as a **P0 fix** before launch.

### Migrated blog and guide content

Scraped WordPress HTML often has:

- **Missing or weak alt text** on decorative images
- **Lazy-load placeholders** (`src` empty SVG, real image in `data-lazy-src`)—bad for no-JS and some assistive tech until fixed during image migration
- **Heading hierarchy skips** (multiple H2s, no logical H1 in body because the template supplies the title—usually OK if one H1 exists in the template)
- **`aria-level` on `<li>`** from Divi exports—not harmful alone but noisy
- **Color contrast** relying on Divi greens/grays in inline styles—may fail WCAG on some combinations

Do not bulk-rewrite all posts; fix high-traffic URLs and patterns you touch for other QA.

### Keyboard and dropdown behavior

Desktop dropdowns open on **hover** (`:focus-within` also applies). Keyboard users tabbing into a menu item may need Enter/Space on the trigger—verify triggers are real `<button>` elements (they are in the current header).

### Iframes (conversion)

- **Book a call:** LeadConnector iframe—confirm it has a **`title`** attribute describing the booking widget (add in `book-call.astro` if missing).
- **Support center:** ClickUp form iframe—same.

Screen readers use iframe titles to announce what the embedded region is.

### Third-party embeds

YouTube podcast embeds load third-party players. No cookie-consent or “load embed on click” pattern exists today—common tradeoff; note for privacy/accessibility policies if required.

### Motion and animation

Divi and homepage sections may use motion (scroll effects, carousels). No `prefers-reduced-motion` handling is documented sitewide. If animations cause vestibular issues, add reduced-motion CSS on flagship pages first.

### Focus visibility

Custom green pill nav and inline styles may suppress visible focus outlines. Tab through `/book-call/` and homepage CTAs; if focus is invisible, add `:focus-visible` outlines in `global.css` or component styles.

---

## Testing procedure (pre-launch spot-check)

### 1. Keyboard walk (30–45 minutes)

On **staging**, with a browser only (no mouse):

| Page | What to do |
|------|------------|
| `/` | Tab from top; open Practice Areas / Services / Resources; reach Book a Call |
| `/book-call/` | Tab to iframe; confirm you can reach footer |
| `/blog/` (index) | Tab into article list links |
| One blog post | Tab through article links |
| `/support-center/` | Tab to iframe |
| Mobile width | Repeat header menu test |

Pass: every interactive control reachable; no keyboard trap except inside iframe (expected).

### 2. Screen reader sample (optional, 30 minutes)

VoiceOver (Mac) or NVDA (Windows) on:

- Homepage hero and primary CTA
- Book-call page (listen to how iframe is announced)

Pass: page title announced; landmarks sensible; iframe has a name.

### 3. Automated scan (optional, 1 hour setup)

Tools: [axe DevTools](https://www.deque.com/axe/devtools/) browser extension, or `axe-core` in Playwright against staging URLs.

Suggested URLs:

- `/`
- `/book-call/`
- `/about-us/`
- `/privacy-policy/`
- One blog post, one city page

Triage **critical** and **serious** issues only for launch; log the rest.

### 4. Legal and policy pages

Confirm footer links resolve:

- `/privacy-policy/`
- `/terms-of-service/`

Content accuracy is legal/compliance; broken links are an accessibility and trust issue.

---

## Content author guidelines (when editing `.astro`)

When you add or clean up a page:

- **Images:** Always set `alt`. Decorative images use `alt=""`.
- **Headings:** One logical outline; don’t skip levels without reason.
- **Links:** Link text should describe the destination (“Sabbeth Law case study”), not “click here.”
- **Buttons vs links:** Use `<button>` for actions that don’t navigate; `<a href>` for navigation.
- **Forms:** Native labels on any custom form fields (most forms are iframes here).
- **Video/audio:** Provide a text alternative or transcript where required (podcast show notes help).

---

## Relationship to performance and CSS

The site loads a large legacy Divi stylesheet on every page. Accessibility overlaps with performance (zoom, reflow, contrast). A future CSS diet may improve contrast consistency if tokens are centralized—see performance/CSS docs. Not a blocker for the spot-check gate unless Lighthouse accessibility score is a stated business requirement.

---

## Sign-off suggestion

For Phase 4, record something like:

| Check | Pass? | Notes |
|-------|-------|-------|
| Keyboard: header + book-call | | |
| Mobile menu usable | | |
| Iframe titles on book-call + support | | |
| Footer legal links | | |
| axe scan on P0 URLs (if run) | | Critical issues = 0 or waived |

Full WCAG audit: **deferred** unless stakeholder requires otherwise.

---

## Who to involve

- **Dev:** Focus styles, mobile menu, iframe titles, template fixes
- **QA:** Keyboard and spot-check pages on staging
- **Marketing / legal:** Policy page content, any client-facing accessibility claims
- **External auditor:** Only if contract or risk requires formal compliance
