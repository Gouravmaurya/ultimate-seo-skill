---
name: seo-codebase
description: >
  Repo-first SEO: analyze a project's source code (framework, routes, dynamic
  page counts, metadata, schema, sitemaps, robots, rendering, crawl cost), write
  a project SEO profile and scored audit, then implement the fixes in the code
  with framework-native patterns. Use when the user is inside a codebase and
  says "do SEO for this project", "fix SEO in the code", "improve SEO", "add
  metadata/schema/sitemap", "make this site rank", or runs /seo with no URL.
user-invocable: true
argument-hint: "[path] [audit|fix|verify]"
license: MIT
metadata:
  author: Gouravmaurya
  version: "2.5.0"
  category: seo
---

# Codebase SEO: Analyze the Project, Then Improve It

Other sub-skills audit a live URL. This one works on the **source code** and
ships the fixes. Always in this order: **analyze → audit → plan (approval) →
implement → verify.** Never write SEO code before the analysis is done.

Reference files (load when the phase says so):
- `references/page-rules.md`: the page-level standard every indexable page must meet.
- `references/framework-recipes.md`: how to implement each rule in Next.js, Astro, Nuxt, SvelteKit, Remix/React Router, Gatsby, WordPress, static generators, and SPAs.
- For thousands of generated pages, hand off to `seo-indexing`.
- For the answer layer (AI Overviews, AI Mode, ChatGPT, Perplexity), use `seo-aeo` criteria.

## Rules

1. **Evidence only.** Every finding cites `file:line` or a URL you actually fetched. No invented search volumes, rankings, reviews, authors, or statistics. Label estimates.
2. **Claims must match code.** Titles, descriptions, schema and copy may only promise what the product does. If marketing copy promises a feature the code lacks, report it as a finding. It is a trust and E-E-A-T problem, not a copy tweak.
3. **Use the stack's own idioms and helpers.** Extend the project's existing metadata utils, schema builders and sitemap routes. Never add a parallel system or a new dependency when the framework already does it.
4. **Stop for approval before editing code** (Phase 3). Then work in small, reviewable steps and run the project's build/lint/typecheck after each group.
5. **Don't break the product.** SEO changes must not change auth, pricing, data writes, or user flows. When a fix touches rendering or caching, explain the trade-off first.
6. **Outputs go in `docs/seo/`.** Update existing files instead of creating duplicates. On re-runs, read `docs/seo/` first and only re-audit what changed (`git log` since the last audit date).

---

## Phase 1: Analyze the project (read-only)

Record each item with file paths. Read before you grep; trace real flows.

| What | How to find it |
|---|---|
| Framework & version | `package.json`, `composer.json`, `Gemfile`, `go.mod`, config files. Map to a recipe in `framework-recipes.md`. |
| Rendering model per route | SSG / SSR / ISR / client-only. **Client-only rendering of indexable content is a Critical finding**: many AI crawlers do not run JavaScript. |
| Hosting & quotas | `vercel.json`, `netlify.toml`, `wrangler.toml`, Dockerfile, CI. Note free-tier limits (function invocations, image transforms, DB/API quotas) that crawl traffic can exhaust. |
| Canonical domain | `metadataBase`, `site` config, env files, sitemap, README. Check www vs apex and trailing-slash consistency. |
| Route inventory | Every public route: static or dynamic, indexable or private, in sitemap or not. |
| **Dynamic page counts** | For each dynamic route, count real records: DB (only through an existing read-only script or ORM count in a scratch script the user approves), CMS, JSON/TS/MD data. If you cannot count safely, ask. |
| Existing SEO layer | Metadata helpers, `<head>` management, JSON-LD builders, sitemap(s), robots, canonical logic, OG images, `llms.txt`, redirects, 404 handling, hreflang. |
| Index gating | How each dynamic route decides `index` vs `noindex`. **Does the sitemap query use the same rule?** |
| Crawl cost per hit | Does a public render call auth/cookies/headers, run DB writes, paid APIs, AI calls, or image transforms? Is it cached? |
| Internal linking | Header/footer on every template, hubs, breadcrumbs, related-item blocks. Are dynamic pages reachable by `<a href>` links, not only the sitemap? |
| Measurement | GA4/GTM/PostHog/Matomo, Search Console verification, Bing Webmaster verification. |
| Prior work | Existing `docs/seo/`, audits, keyword files. Don't redo finished work. |
| Business context | What the site sells, to whom, where, the main conversion. From README, homepage copy, pricing. Ask only for what you cannot find. |

Write **`docs/seo/PROJECT-SEO-PROFILE.md`**: stack, rendering, domain, route inventory table (route, type, rendering, page count, indexable, in sitemap, cached), business context, existing SEO assets, open questions.

## Phase 2: Audit (scored)

Load `references/page-rules.md`. For the areas the other sub-skills own, apply their criteria to the code: `seo-technical`, `seo-schema`, `seo-sitemap`, `seo-content`, `seo-geo`, `seo-aeo`, `seo-programmatic`, `seo-hreflang`, `seo-images`. If a production URL exists, you may also run the URL-based sub-skills and compare live output with the code.

Write **`docs/seo/SEO-AUDIT-YYYY-MM-DD.md`** with a 0-100 score per area plus an overall score using the orchestrator's weights:

1. Indexation (sitemaps, robots, noindex, canonicals, status codes)
2. Rendering & crawlability (SSR/SSG, links, JS dependence)
3. On-page (titles, descriptions, H1, headings, one intent per page)
4. Content & E-E-A-T (thin pages, authorship, sources, first-hand data, claims vs code)
5. Structured data (valid, honest, current, no retired types)
6. Performance (CWV risks visible in code: image sizing, fonts, JS weight, LCP element)
7. AI search readiness (see `seo-aeo` and `seo-geo`)
8. Internal linking
9. Crawl cost & scale (only when a dynamic route exceeds about 500 pages; see `seo-indexing`)

Each finding carries: `file:line`, severity (Critical/High/Medium/Low), the fix, and **how we'd know it worked** (a check the user can run). End with **Top 5 critical issues** and **Top 5 quick wins**.

## Phase 3: Action plan (stop for approval)

Write **`docs/seo/ACTION-PLAN.md`**: a table with item, finding it fixes, impact (H/M/L), effort (S/M/L), files, and a verification check. Order the items like this:

1. Anything that blocks indexing (noindex leaks, robots blocks, 5xx, client-only content, wrong canonicals).
2. Crawl-cost risks (uncached dynamic renders, paid calls on crawler hits).
3. Quick wins (titles, descriptions, canonicals, schema fixes).
4. Structure (hubs, breadcrumbs, internal links, sitemaps).
5. Content (answer-first rewrites, E-E-A-T, new pages from keyword gaps).

Show the top items and **ask which to implement.** Do not edit code before the user answers.

## Phase 4: Implement

For each approved item:
- Load the matching recipe from `framework-recipes.md`; follow `page-rules.md` for the standard.
- Reuse existing helpers. Put new shared logic (metadata builder, JSON-LD builder, index gate) in one module that every template imports.
- Generated routes: follow `seo-indexing` (one index gate shared by page and sitemap, cached rendering, split sitemaps, internal links).
- Run build/lint/typecheck after each group. Tick the item off in `ACTION-PLAN.md`.

## Phase 5: Verify against real output

Code review is not verification. Start the dev server or build (or fetch production), then check **one page of every route type**:

- `<title>`, meta description, canonical, robots meta, `og:*` all present in the **server HTML** (view-source or `curl`, not the DOM after hydration).
- Exactly one `<h1>`; JSON-LD parses as JSON and matches visible content.
- Missing records return a real **404** status; redirects are single-hop **301/308**.
- Every sitemap returns valid XML; spot-check 10 URLs per sitemap: 200, indexable, self-canonical.
- `robots.txt`: private paths are disallowed in **every** user-agent group (a bot obeys only its most specific group).
- Content is present without JavaScript (`curl` the page and grep for the main copy).

List what the user must do by hand: submit sitemaps in Search Console and Bing Webmaster Tools, run the Rich Results Test on changed templates, and set up IndexNow keys if chosen.

## Output

- `docs/seo/PROJECT-SEO-PROFILE.md`
- `docs/seo/SEO-AUDIT-YYYY-MM-DD.md`, containing the **Codebase SEO Score: XX/100** and per-area scores
- `docs/seo/ACTION-PLAN.md` (kept current)
- The code changes, with a short summary: what changed, before/after scores, manual follow-ups.

## Error Handling

| Scenario | Action |
|---|---|
| Framework not recognized | Report the signals found, apply `page-rules.md` directly, and ask the user how pages are rendered. |
| Cannot count dynamic records safely | Report the route as "count unknown", ask the user for a count, and do not guess. |
| Build fails after a change | Revert that change, report the error, and continue with the next item only if it is independent. |
| No production URL | Verify against the local dev server or build output; say that live checks (Rich Results Test, Search Console) remain for the user. |
| Monorepo | Profile each deployable app separately; ask which app to work on first. |
