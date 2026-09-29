# Page Rules: The Standard for Every Indexable Page

Framework-neutral rules. For how to implement each one in a given stack, see
`framework-recipes.md`. Facts verified 2026-09-29 against Google Search Central,
OpenAI, and Anthropic documentation (sources at the end).

---

## 1. Honesty (applies to everything below)

- Titles, descriptions, schema, and copy state only what the product actually does and the data actually shows.
- No invented ratings, reviews, authors, credentials, statistics, or "as seen in" claims.
- Schema describes **visible** content. Markup that doesn't match the page is a spam-policy risk, not an optimization.
- Programmatic pages built from third-party data must add something of their own (computed metrics, analysis, local context). Republishing a feed with light rewording is **scaled content abuse** (Google spam policies, updated 2026-08-28).

## 2. Metadata

### Title (`<title>`)
- Unique per page; primary intent first, brand last. About 50-60 characters is a display heuristic, not a rule; Google may rewrite titles.
- Detail page: `{Entity} in {Place}: {Key fact} | {Brand}`.
- Category/hub: `{Category} in {Place} ({count} options) | {Brand}`. Use a count only if it's real and current.
- Comparison: `{A} vs {B}: {What differs} ({Year}) | {Brand}`. Use this only when the page has a genuine comparison.

### Meta description
- Unique; about 140-160 characters; says what the visitor gets on this page. Google may substitute page text.
- Programmatic pages: build it from real fields (price, size, place, key metric). Don't truncate a source description that is duplicated across other sites.

### Canonical
- Every indexable page has one absolute, self-referencing canonical, in the **server HTML**.
- One canonical per page (conflicting canonicals cause indexing issues).
- Strip tracking params; pick one host (www or apex), one protocol (https), one trailing-slash style, and redirect the rest with a single 301/308.

### Robots meta
- Default: indexable. Use `noindex, follow` for thin/incomplete generated pages, internal search results, and filtered duplicates.
- Put `noindex` in the **initial HTML**. Google may skip rendering when it sees `noindex`, so a JS-removed `noindex` can keep a page out.
- Snippet controls (`nosnippet`, `max-snippet`, `data-nosnippet`) also govern appearance in AI Overviews and AI Mode.

### Social (Open Graph / Twitter)
- `og:title`, `og:description`, `og:url`, `og:type`, `og:image` (absolute URL, about 1200×630, with `og:image:alt`), `twitter:card=summary_large_image`.

## 3. Structured Data (JSON-LD)

- Inject as `<script type="application/ld+json">` in server HTML. When serializing, escape `<` (`JSON.stringify(data).replace(/</g, '\\u003c')`) so page data can't break out of the script tag.
- Google's AI optimization guide says structured data is **not required** for generative AI search. Use it for rich-result eligibility and entity clarity, not as an AI "hack".

| Page type | Use | Notes |
|---|---|---|
| Site-wide (home) | `Organization` (logo, `sameAs` to real profiles), `WebSite` | One canonical entity; same name everywhere. |
| Article / guide | `Article` / `BlogPosting` with real `author` (`Person` or the `Organization`), `datePublished`, `dateModified` | `dateModified` changes only when the content does. |
| Hierarchical pages | `BreadcrumbList` | Mirrors the visible breadcrumb. |
| Product / listing | `Product` + `Offer` (`price`, `priceCurrency`, real `availability`) | Sold/out of stock ≠ `InStock`. |
| Local business | The most specific `LocalBusiness` subtype | NAP identical to Google Business Profile. |
| Genuine user Q&A | `QAPage` | For one question with user answers. |
| Software / tool | `SoftwareApplication` / `WebApplication` | `offers` only if pricing is shown. |

**Retired or restricted (don't recommend):**
- **FAQ rich results**: no longer shown in Google Search since **2026-05-07**. `FAQPage` stays valid Schema.org and Google still parses it, but don't add it for a SERP feature. Existing markup: leave it unless it's inaccurate.
- **HowTo rich results**: deprecated (2023).
- `AggregateRating` / `Review` only for genuine first-party reviews visible on the page. Self-serving reviews on your own `Organization`/`LocalBusiness` aren't eligible.

## 4. Content & Semantics

- Exactly one `<h1>` that states the page's primary intent; `<h2>`/`<h3>` in order, never used just for styling.
- `<main>`, `<nav aria-label>`, `<article>`, `<time datetime>` where they fit.
- **Answer first**: the first 1-2 sentences under a heading answer that heading's question. Write for people; Google's guide says not to "chunk" or rewrite content just for AI.
- Visible "last updated" dates on content that changes; a named author with a bio on advice content.
- Cite primary sources for statistics and claims (see `seo-aeo`).

## 5. Images

- Descriptive `alt` on every meaningful image (`alt=""` for decorative ones).
- Explicit `width`/`height` or `aspect-ratio` so images don't shift the layout (CLS).
- LCP/hero image: eager + `fetchpriority="high"`; everything below the fold: `loading="lazy"`.
- Modern formats (AVIF/WebP) with responsive `srcset`.
- Hosted image optimization has quotas. Thousands of crawlable pages with remote photos can exhaust a free tier, so pre-size images or serve originals for remote images (see `seo-indexing`).

## 6. Links & Crawlability

- Internal links are `<a href="…">` with descriptive anchor text. Google discovers links only from `<a>` elements with `href`, so `onClick` navigation and URL fragments are invisible to it.
- Every template (including detail pages) includes the site header/footer, which link back to hubs.
- Pagination uses crawlable links (`?page=2`), not only infinite scroll.
- Indexable content is in the server HTML. Many AI crawlers fetch raw HTML without executing JavaScript.

## 7. Status Codes

- Missing record → **404** (or **410** if permanently gone), never a 200 "not found" page (soft 404).
- Moved → single-hop **301/308** to the final URL. No redirect chains.
- Private pages → 401/403 or login redirect, plus `Disallow` in robots.txt.
- Support conditional requests (`ETag`/`Last-Modified` → 304) where the host allows it. Google's crawl-budget guide recommends it.

## 8. robots.txt

- Disallow private paths (`/api/`, `/admin/`, `/dashboard/`, auth, checkout, internal search).
- List every sitemap with absolute `Sitemap:` lines.
- **Group-override trap:** a crawler obeys only its most specific `User-agent` group. A separate `User-agent: GPTBot` / `Allow: /` block ignores all `Disallow` lines under `*`. Put named bots in the same group as `*`, or repeat the `Disallow` lines.
- `robots.txt` controls crawling, not indexing. To keep a page out of the index, let it be crawled and use `noindex`.

### AI crawlers (verified 2026-09-29)

| Purpose | User-agent | Notes |
|---|---|---|
| Google Search, AI Overviews, AI Mode | `Googlebot` | Blocking it removes you from Search. |
| Gemini/Vertex training & grounding | `Google-Extended` | Does **not** affect Search or AI Overviews. |
| ChatGPT search results | `OAI-SearchBot` | Blocking it drops you from ChatGPT search answers. About 24 hours to take effect. |
| OpenAI model training | `GPTBot` | Independent of `OAI-SearchBot`. |
| ChatGPT user-initiated fetches | `ChatGPT-User` | robots.txt "may not apply". |
| Claude search results | `Claude-SearchBot` | Honors robots.txt. |
| Anthropic model training | `ClaudeBot` | Honors robots.txt. |
| Claude user-initiated fetches | `Claude-User` | Honors robots.txt. |
| Perplexity search | `PerplexityBot` | |
| Bing, Copilot, and much of ChatGPT's retrieval | `Bingbot` | Bing index quality matters for ChatGPT citations. |

`anthropic-ai` and `Claude-Web` are **not** in Anthropic's current documentation. Don't rely on them.

To be cited in AI search, allow the **search** bots (`Googlebot`, `OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`, `Bingbot`). Allowing or blocking the **training** bots is a licensing choice, not a visibility one.

**SEO-tool scrapers** (`AhrefsBot`, `SemrushBot`, `MJ12bot`, `DotBot`, `PetalBot`) bring no search traffic, and each hit is a full render. On large dynamic sites, disallow the ones you don't use. Site operators have reported `Bytespider` ignoring robots.txt; if it matters, block it at the CDN/WAF as well.

## 9. Sitemaps

- Only canonical, indexable, 200-status URLs. Never noindexed, redirected, or blocked URLs.
- At most 50,000 URLs or 50 MB uncompressed per file; split by type, then use a sitemap index.
- `<lastmod>` = last **significant content** change, accurate and verifiable. Google ignores `<changefreq>` and `<priority>`.
- Submit via Search Console (UI or API) and `Sitemap:` lines in robots.txt. Google's old sitemap "ping" endpoint no longer works; don't build on it.

## 10. Crawl Cost (dynamic sites)

A page that is cheap to crawl is a page you can afford to have indexed:
1. Cache public renders (SSG/ISR/CDN); render on first request, then serve from cache.
2. No per-request auth/cookies/headers or per-viewer work in the public render. Load those client-side.
3. No DB writes, paid API calls, or AI calls on crawler or anonymous hits. Cap retries on paid calls.
4. Revalidate the cached page when its data changes.
5. Full details: `seo-indexing`.

## 11. AI Search Readiness (summary; details in `seo-aeo` / `seo-geo`)

- Content in server HTML, search bots allowed, snippets allowed.
- Answer-first sections, specific facts with cited sources, visible dates, clear entity (Organization/Person with `sameAs`).
- `llms.txt` is optional: Google says Search ignores it; other AI systems may read it. Never present it as a ranking lever.

## 12. Pre-Launch Checklist

- [ ] Unique title and description, from real data, with no overclaims
- [ ] One self-referencing absolute canonical in server HTML
- [ ] Correct robots meta (thin/incomplete → `noindex, follow`)
- [ ] One `<h1>`; ordered headings; answer-first intro
- [ ] JSON-LD valid, visible-content-matching, no retired types for SERP features
- [ ] OG/Twitter tags with an absolute 1200×630 image
- [ ] Images: alt, dimensions, lazy below the fold, priority hero
- [ ] Internal links are `<a href>`; header/footer present
- [ ] Missing → 404/410; redirects single-hop
- [ ] In the right sitemap (if indexable); not blocked in any robots group
- [ ] Public render cached; no paid calls on crawler hits (dynamic routes)
- [ ] Verified in the rendered server HTML, not just in code

---

## Sources (checked 2026-09-29)

- Google, *Optimizing for generative AI features on Google Search* (updated 2026-07-10): developers.google.com/search/docs/fundamentals/ai-optimization-guide
- Google, *Spam policies* (updated 2026-08-28): developers.google.com/search/docs/essentials/spam-policies
- Google, *JavaScript SEO basics* (updated 2026-03-04): developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics
- Google, *Build and submit a sitemap* (updated 2026-07-08): developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap
- Google, *Crawl budget management* (updated 2026-07-22): developers.google.com/crawling/docs/crawl-budget
- Google FAQ rich result retirement (2026-05-07): Search Engine Journal, "Google Drops FAQ Rich Results From Search"
- OpenAI, *Overview of OpenAI crawlers*: developers.openai.com/api/docs/bots
- Anthropic, *Does Anthropic crawl data from the web…*: support.claude.com/en/articles/8896518
