---
description: SEO Master — audit and fix SEO across the entire project (Next.js, Flutter web, or any web app)
argument-hint: "[page|component|all]"
---

# SEO Master

Full SEO audit and fix pipeline. Detects framework, analyzes every page/screen, scores SEO health, then applies fixes automatically.

---

## Quick Reference

```bash
/seo-master           # Full project SEO audit + auto-fix
/seo-master audit     # Audit only, no changes
/seo-master fix       # Apply fixes from last audit
/seo-master [page]    # Audit specific page or route
```

---

## Phase 1: Detect Project Type

Inspect the root directory:
- `pubspec.yaml` → Flutter web
- `next.config.*` → Next.js
- `package.json` + Vite/React → SPA
- Other → generic HTML/web

Report detected framework before proceeding.

---

## Phase 2: Audit

### 2.1 Meta & Head Tags
Check every page/route for:
- `<title>` — present, unique, 50–60 chars
- `<meta name="description">` — present, unique, 150–160 chars
- `<meta name="keywords">` — present (optional but check)
- Open Graph: `og:title`, `og:description`, `og:image`, `og:url`
- Twitter Card: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- `<link rel="canonical">` on every page

**Next.js**: check `metadata` exports, `generateMetadata()`, `<Head>` usage.
**Flutter web**: check `web/index.html` and any dynamic meta injection.

### 2.2 Structured Data (Schema.org)
- JSON-LD present for appropriate page types (Article, Product, FAQ, BreadcrumbList, Organization)
- Valid schema — no missing required fields
- Nested schemas correctly formed

### 2.3 Semantic HTML
- Single `<h1>` per page
- Heading hierarchy: h1 → h2 → h3 (no skips)
- Images have `alt` attributes (non-empty, descriptive)
- Links have descriptive text (no "click here" or bare URLs)
- Landmark elements: `<main>`, `<nav>`, `<header>`, `<footer>`, `<article>`, `<section>`

### 2.4 Performance & Core Web Vitals
- Images: `next/image` or lazy-loading attributes present
- Font loading: `font-display: swap` set
- No render-blocking scripts without `defer` or `async`
- Above-the-fold content prioritized (no lazy on hero images)

### 2.5 Crawlability
- `robots.txt` exists and is correctly configured
- `sitemap.xml` exists and includes all public routes
- No `noindex` on pages that should be indexed
- Internal links use relative paths or correct domain

### 2.6 URL Structure
- URLs are lowercase, hyphen-separated
- No query-string-only pages for main content
- Dynamic routes have proper slugs

### 2.7 Mobile & Accessibility
- `<meta name="viewport">` present
- Touch targets ≥ 44px
- Color contrast meets WCAG AA (4.5:1 for text)

---

## Phase 3: Score

For each page, produce a score table:

```
Page: /about
┌─────────────────────────┬────────┬──────────────────────────┐
│ Check                   │ Status │ Issue                    │
├─────────────────────────┼────────┼──────────────────────────┤
│ Title tag               │  PASS  │ -                        │
│ Meta description        │  FAIL  │ Missing                  │
│ Open Graph              │  WARN  │ og:image missing         │
│ Structured data         │  PASS  │ -                        │
│ Heading hierarchy       │  FAIL  │ Two <h1> tags found      │
│ Image alt texts         │  WARN  │ 3 images missing alt     │
│ Canonical               │  FAIL  │ Missing                  │
│ robots.txt              │  PASS  │ -                        │
│ sitemap.xml             │  PASS  │ -                        │
└─────────────────────────┴────────┴──────────────────────────┘
SEO Score: 62/100
```

Aggregate score across all pages at the end.

---

## Phase 4: Fix (unless `audit` argument passed)

Apply all FAIL and WARN fixes automatically:

1. Add missing meta tags in the correct framework pattern
2. Generate or update Open Graph tags
3. Add JSON-LD structured data appropriate to page type
4. Fix heading hierarchy (promote/demote headings minimally)
5. Add `alt` attributes to images (use filename/context to generate descriptive text)
6. Create `robots.txt` if missing
7. Generate `sitemap.xml` from discovered routes if missing
8. Add `<link rel="canonical">` where absent

**Never change page content or user-facing copy without explicit confirmation.**

---

## Phase 5: Report

```markdown
# SEO Master Report — {date}

**Project**: {framework}
**Pages audited**: {N}
**Overall score**: {X}/100

## Fixed
- [x] Added meta description to /about, /contact
- [x] Generated sitemap.xml with 12 routes
- [x] Added OG tags to all pages
- [x] Fixed heading hierarchy on /blog/[slug]

## Warnings (manual review needed)
- [ ] /shop — og:image is a placeholder, replace with real product image
- [ ] /landing — title is 72 chars, consider shortening

## Remaining Issues
- [ ] Structured data for /product pages needs real product data from API
```

---

## Framework-Specific Notes

**Next.js (App Router)**
- Use `export const metadata` or `generateMetadata()` in `page.tsx`
- Use `next/image` for all images
- `robots.ts` and `sitemap.ts` in `app/` directory

**Next.js (Pages Router)**
- Use `next/head` with `<Head>` component
- `public/robots.txt` and `public/sitemap.xml`

**Flutter Web**
- Meta tags go in `web/index.html`
- Dynamic meta requires JS injection or pre-rendering
- Flag if SSR/pre-rendering is not set up — SEO will be limited

**SPA (Vite/React)**
- Warn if no SSR/SSG — Google crawls SPAs but it is unreliable
- Recommend adding a prerender step or switching to Next.js for SEO-critical pages
