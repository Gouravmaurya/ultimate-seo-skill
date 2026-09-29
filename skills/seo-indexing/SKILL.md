---
name: seo-indexing
description: >
  Get hundreds to millions of pages crawled and indexed: index-gate and sitemap
  parity, crawl cost, sitemap splitting, internal-link discovery, staged
  rollout, submission (Search Console, Bing, IndexNow), lifecycle of
  sold/removed pages, and diagnosis of every Search Console "not indexed"
  reason. Works from the codebase and/or a live URL. Use when user says
  "index my pages", "5000 pages", "pages not indexed", "crawled currently not
  indexed", "discovered not indexed", "crawl budget", "IndexNow", "submit
  sitemap", or "bulk indexing".
user-invocable: true
argument-hint: "[plan|diagnose|implement] [url or path]"
license: MIT
metadata:
  author: Gouravmaurya
  version: "2.5.0"
  category: seo
---

# Bulk Indexing: From "Discovered" to "Indexed" at Scale

Built from a real rollout: pushing about 5,000 property-listing pages through a
dedicated sitemap on a young domain, then fixing what that load broke (quota
exhaustion, uncached renders, sitemap/noindex mismatch). Complements
`seo-programmatic` (template quality and uniqueness gates) and `seo-sitemap`
(sitemap validation). This skill owns **getting the pages indexed and keeping
crawling affordable**.

## Set expectations first

Tell the user plainly:
- A sitemap gets URLs **discovered**, not **indexed**. Google indexes what it judges unique and useful, at a pace set by site quality, popularity and server health.
- Google's crawl-budget guide (updated 2026-07-22) addresses **1M+ page sites updating weekly, 10K+ page sites changing daily, or sites with many "Discovered – currently not indexed" URLs**. Below that, indexing problems are almost always **quality or discovery** problems, not budget.
- **The Google Indexing API is only for pages with `JobPosting` or `BroadcastEvent` (in a `VideoObject`).** Using it for listings, products, or articles breaks its terms and can get access revoked. Never recommend it as a shortcut.
- Never promise "all N pages indexed" or a timeline.

## Modes

| Mode | Input | Output |
|---|---|---|
| `plan` | codebase and/or URL | `docs/seo/INDEXING-PLAN.md` |
| `diagnose` | Search Console export, `gsc_inspect.py` output, or the user's description of the Pages report | `docs/seo/INDEXING-DIAGNOSIS.md` |
| `implement` | codebase (after `plan` is approved) | code changes + updated plan |

If Google API credentials exist (`"${CLAUDE_PLUGIN_ROOT}/scripts/claude-seo" run google_auth.py --check`), use `seo-google` (`gsc_inspect.py`, `gsc_query.py`) for real index status instead of asking the user.

---

## Step 1: Size and segment

For each generated route: total records, records passing the index gate, how often data changes, and the cost of one render (DB queries, API calls). Then pick an approach:

| Indexable URLs | Approach |
|---|---|
| < 500 | One sitemap; `seo-programmatic` quality gates; normal page rules. |
| 500 - 50,000 | A sitemap per page type, cached rendering **required**, staged rollout. |
| > 50,000 | Chunked sitemaps + sitemap index, strict gate, rollout by segment, crawl-stats monitoring. |

Segment by value (for example top markets, complete records, in-stock items). You'll roll out best-first.

## Step 2: One index gate, used by page and sitemap

Write **one** function that decides whether a record deserves indexing (has the required fields plus enough unique content). Import it in **both** the page's robots meta and the sitemap query. Express it as a DB `where` clause if possible, so the sitemap query stays cheap.

If the two differ, the sitemap submits URLs the page marks `noindex`. That shows up in Search Console's Pages report as "URL marked 'noindex'" for submitted URLs and wastes crawls.

> **Real case:** the sitemap route filtered `price > 0` plus address fields, while the page's `isCompleteListing()` required `price >= 10000` and a title. Every listing in between was submitted and then told "noindex". Fix: export the gate and reuse it.

Also make sure:
- Missing records return **404/410**, not a 200 "not found" (a soft 404).
- The gate excludes near-duplicates (same entity under two IDs, variant URLs). Pick one canonical.

## Step 3: Make each page worth indexing

Re-run `seo-programmatic`'s uniqueness gates. At scale, the page must carry something only you have:
- computed data (e.g. per-listing cap rate, cash flow, price history, score),
- context (local market stats, comparisons with nearby items),
- a title/description built from real fields, not a truncated source description shared with other sites.

Republishing a feed with light rewording is **scaled content abuse** under Google's spam policies (updated 2026-08-28). Lots of "Crawled – currently not indexed" usually means this step is failing.

## Step 4: Make crawling cheap (before submitting)

Thousands of URLs means constant bot traffic: Googlebot, Bingbot, AI search bots, SEO-tool scrapers. If each hit costs a server render, DB queries, or paid calls, crawling burns quotas and money, and slow or failing responses **lower Google's crawl capacity** for the site.

1. **Cache the public render** (SSG/ISR/CDN; see `seo-codebase/references/framework-recipes.md`). Render on first request, serve from cache, and revalidate when data changes.
2. **Keep per-request APIs out of the public render** (`cookies()`, `headers()`, session/auth). They force dynamic rendering. Load per-viewer state client-side.
3. **No DB writes on bot hits.** Record views/analytics client-side, filtered with a bot user-agent check.
4. **No paid enrichment, third-party API, or AI calls in anonymous/cached renders.** Record the attempt time and don't retry a record with no data for about 30 days. Cap retries: model × key × retry loops multiply one failure into dozens of billed calls.
5. **Image optimization quotas:** hosted transforms (for example Vercel's metered image optimization) run out fast across thousands of pages with remote photos. Pre-size images or serve remote originals, and document the trade-off.
6. **Return 304s** for unchanged pages where the host supports `ETag`/`Last-Modified` (Google's crawl-budget guide).
7. **Disallow SEO-tool scrapers you don't use** in robots.txt (`AhrefsBot`, `SemrushBot`, `MJ12bot`, `DotBot`, `PetalBot`), and put named AI bots in the same group as `*` so private-path `Disallow` lines apply to them too.

## Step 5: Sitemaps that scale

- One sitemap per page type (for example `sitemap.xml` for static and editorial pages, `sitemap-properties.xml` for listings). Each file: **≤ 50,000 URLs and ≤ 50 MB uncompressed**. Above that, chunk the files and add a sitemap index.
- Next.js: `generateSitemaps()` + `sitemap({ id })`, where in v16 `id` is a `Promise<string>`. Astro: `@astrojs/sitemap` (auto-index). Nuxt: `@nuxtjs/sitemap`. Others: a route that streams XML per chunk plus an index.
- `<lastmod>` = last **content** change (e.g. `priceChangedAt ?? updatedAt`), never request time. Google uses it only when "consistently and verifiably accurate", and it ignores `changefreq` and `priority`.
- Cache the sitemap response (it's often the heaviest query on the site). In Next.js, `dynamic = 'force-dynamic'` overrides `revalidate`; rely on `Cache-Control: s-maxage` or drop `force-dynamic`.
- If you cap the count, order by value/freshness so the best pages go first.
- List every sitemap (or the index) in robots.txt.

## Step 6: Internal links (discovery beats sitemaps)

Pages only in a sitemap get crawled less than pages linked from pages Google already crawls.
- Hubs link to detail pages: category/city/market pages list items, with **crawlable pagination** (`<a href="?page=2">`).
- Each detail page links to 4-8 siblings ("similar", "nearby", "same category") and up to its hubs through breadcrumbs.
- Detail templates include the site header/footer.
- Link new hubs from the pages that get crawled most (homepage, top posts, top landing pages).
- Google discovers links only from `<a href>`, not `onClick` or fragment routes.

## Step 7: Staged rollout

1. Open the best segment first (e.g. 300-1,000 complete, high-value pages).
2. Submit the sitemap. Wait **2-3 weeks** and read Search Console → Pages, **filtered by that sitemap**.
3. If most pages are indexed and few are "Crawled – currently not indexed", widen the gate to the next segment.
4. If many are "Crawled – currently not indexed", Google sees low value. Improve the template (Step 3) before adding URLs.
5. If many are "Discovered – currently not indexed", improve internal links and response speed (Steps 4 and 6), then wait.

## Step 8: Submission channels (manual steps for the user)

| Channel | What to do | Notes |
|---|---|---|
| Google Search Console | Submit each sitemap (or the index) in the Sitemaps report or via the API. | "Request indexing" in URL Inspection is for a handful of key pages only (small daily quota). |
| robots.txt | `Sitemap:` lines for every sitemap. | Discovered by all major engines. |
| Bing Webmaster Tools | Import the site from Search Console; submit sitemaps. | Bing's index feeds Copilot and much of ChatGPT search retrieval. Its **AI Performance** report (Feb 2026) shows Copilot citations and grounding queries. |
| IndexNow | Host a key file (`/<key>.txt`; 8-128 characters of letters, digits, hyphens); `POST` up to **10,000 URLs** per request to `https://api.indexnow.org/indexnow` with `{host, key, keyLocation, urlList}`. Handle 200/202 (ok/pending), 400, 403 (bad key), 422 (URL/host mismatch), 429 (too many). | Bing, Yandex, Seznam, Naver and others. **Not Google.** Call it where you revalidate pages after create/update/delete. |
| Google Indexing API | **Don't use** unless pages are JobPosting / livestream BroadcastEvent. | Policy-restricted; abuse can revoke access. |

## Step 9: Lifecycle

- **Sold / out of stock / expired, but useful:** keep the URL at 200, show the status on-page, update schema `availability`, link to similar active items, and revalidate the cache.
- **Gone with no value:** return 404 or 410 and remove it from the sitemap. Google's crawl-budget guide prefers 404/410 over `noindex` for removed pages, because noindexed URLs keep getting crawled.
- **Merged/moved:** single-hop 301/308 to the replacement; update internal links.
- On every change, revalidate the cached page, update `lastmod`, and ping IndexNow.

## Step 10: Monitor

Keep `docs/seo/INDEXING-LOG.md` weekly: URLs per sitemap, submitted vs indexed (per sitemap), top not-indexed reasons with counts, crawl-stats host status, and impressions/clicks for the page type (plus Search Console's **Search generative AI** report impressions, rolled out worldwide 2026-08-31; impressions only, no clicks).

### Search Console reasons → fixes

| Reason | Meaning | Fix |
|---|---|---|
| Discovered – currently not indexed | Found but not crawled yet; Google rescheduled to avoid overloading the site. | More internal links from crawled pages, faster/cached responses, fewer low-value URLs competing for crawls. Then wait. |
| Crawled – currently not indexed | Crawled; Google chose not to index (for now). Google says no resubmission needed. | Treat as a quality signal at scale: add unique data (Step 3), tighten the gate, consolidate near-duplicates. |
| URL marked 'noindex' | Page says `noindex`. | Intended? Remove it from the sitemap. Not intended? Fix the gate (Step 2); make sure `noindex` isn't in the initial HTML by accident. |
| Duplicate without user-selected canonical | Google sees a duplicate with no canonical declared. | Add a self-canonical on the preferred URL; make the pages differ or consolidate them. |
| Duplicate, Google chose different canonical than user | Your canonical disagrees with Google's pick. | Check they're truly different; fix parameters/variants; point internal links and the sitemap at one URL. |
| Alternate page with proper canonical tag | Working as intended. | None. |
| Soft 404 | 200 status with "not found" or empty content. | Real 404/410; or add real content if the page should exist. In SPAs, inject `noindex` on client "not found" views. |
| Not found (404) | 404 when crawled. | Fine if intended. If moved, 301. Remove from the sitemap. |
| Page with redirect | A redirected URL (not indexed itself). | Sitemap and internal links should use final URLs. |
| Redirect error | Chains, loops, overly long URLs. | Single-hop redirects. |
| Server error (5xx) | Render failed or timed out under crawl. | Step 4 (caching, no paid calls); check Crawl Stats host status. |
| Blocked by robots.txt | A `Disallow` matches. | Unblock if it should be indexed. To de-index, allow the crawl and use `noindex` instead. |
| Indexed, though blocked by robots.txt | Indexed from external links despite the block. | To remove: unblock and add `noindex`. |
| Blocked due to unauthorized request (401) / access forbidden (403) | Auth wall or WAF blocking Googlebot. | Make public pages public; allow verified Googlebot in the WAF. |
| Page indexed without content | Content unreadable (cloaking, unsupported format, JS-only). | Serve content in the server HTML; check URL Inspection's rendered view. |

## Output

`INDEXING-PLAN.md`:
1. **Indexing Readiness Score: XX/100** (gate parity 20, page value 20, crawl cost 20, sitemaps 15, internal links 15, submission & monitoring 10)
2. Route table: route, records, pass gate, in sitemap, cached, est. cost per crawl
3. Findings with `file:line` and severity
4. Rollout plan (segments, order, dates, success thresholds)
5. Manual submission checklist
6. Monitoring plan (`INDEXING-LOG.md` template)

`INDEXING-DIAGNOSIS.md`: reason → count → likely cause (with evidence) → fix → how we'd know it worked.

## Error Handling

| Scenario | Action |
|---|---|
| No Search Console data | Plan from the code and live checks; ask the user to export the Pages report (reason + URL examples) for `diagnose`. |
| Record counts unavailable | Ask for counts; don't guess. Mark routes "count unknown". |
| User asks to use the Indexing API for normal pages | Decline and explain the JobPosting/BroadcastEvent restriction; offer the sitemap + internal-link + IndexNow plan instead. |
| Pages fail `seo-programmatic` hard-stop gates | Stop the rollout plan; fix the template first. |
