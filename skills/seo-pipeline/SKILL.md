---
name: seo-pipeline
description: >
  The 3-step SEO pipeline: (1) AUDIT the project and live site and report,
  (2) RESEARCH keywords and competitors and report, (3) IMPROVE, which
  implements the SEO work and reports exactly what changed and what improved.
  Pauses for review after each step and resumes where it left off. Use when
  user says "do SEO for this project", "full SEO", "SEO from start to finish",
  "audit, research and fix", "3 step SEO", or runs /seo pipeline.
user-invocable: true
argument-hint: "[audit|research|improve|status] [url]"
license: MIT
metadata:
  author: Gouravmaurya
  version: "2.6.0"
  category: seo
---

# SEO Pipeline: Audit → Research → Improve

One command, three steps, three reports. Each step reuses the specialist
sub-skills; this skill decides the order, what to hand forward, and what the
reports must contain.

| Step | Command | Output |
|---|---|---|
| 1. Audit | `/seo pipeline audit` | `docs/seo/01-AUDIT-REPORT.md` |
| 2. Research | `/seo pipeline research` | `docs/seo/02-RESEARCH-REPORT.md` + `docs/seo/keyword-map.csv` |
| 3. Improve | `/seo pipeline improve` | code/content changes + `docs/seo/03-IMPROVEMENT-REPORT.md` |

`/seo pipeline` with no step **resumes**: it runs the first step whose report
doesn't exist yet (or is older than 30 days). `/seo pipeline status` lists
which reports exist, their dates, and the next step.

**After each step, stop.** Show a short summary (scores, top findings, what
the next step will do) and ask whether to continue. Never run all three steps
without the user seeing reports 1 and 2.

## Rules for all steps

- Follow the rules of `seo-codebase`: evidence with `file:line` or fetched URL; no invented volumes, rankings, reviews, authors or statistics; claims must match the product; use the stack's own idioms; stop for approval before editing code.
- **Label every number with its source.** Keyword volumes come from Google Search Console, Keyword Planner, DataForSEO, or another named tool; otherwise they're a tier estimate marked "estimated". Never present an estimate as data.
- All files go in `docs/seo/`. Update, don't duplicate. Report dates use `YYYY-MM-DD`.
- Work out the mode first: **codebase** (the project is open), **URL** (a live site is given), or **both** (preferred: the code shows what's intended, the live site shows what Google sees).

## Skill coverage: every skill, every run

The pipeline considers **every sub-skill and extension in this repo**. Each
one is either **run** or **skipped with a stated reason** (not relevant to this
business, or needs credentials that aren't configured). Every report ends with
a **"Skills used"** table (skill → ran / skipped → reason → output file), so
nothing is silently ignored.

| Skill | 1. Audit | 2. Research | 3. Improve | Runs when |
|---|---|---|---|---|
| `seo-codebase` | Phase 1-2: project profile + code audit | page/route inventory for keyword mapping | Phase 3-5: plan, implement, verify | a codebase is open |
| `seo-audit` | full parallel audit of the live site | | final re-score for comparison | a live URL exists |
| `seo-page` | deep check of top 5 landing pages | | verify changed key pages | always (URL or local build) |
| `seo-technical` | crawlability, indexability, security, CWV, IndexNow | | fix technical findings | always |
| `seo-content` | E-E-A-T, thin content, readability | competitor content depth | improve existing copy | always |
| `seo-content-brief` | | brief for each new page / cluster | write new pages from briefs | new pages are planned |
| `seo-schema` | detect + validate markup | competitor schema | add/fix markup | always |
| `seo-sitemap` | validate sitemaps | | generate/fix sitemaps | always |
| `seo-images` | alt, size, format, LCP image | image SERP opportunities | optimize images | always |
| `seo-image-gen` | | | OG/hero images for new/key pages | banana extension installed |
| `seo-geo` | AI crawler access, brand/entity signals | brand mention gaps vs competitors | crawler rules, entity markup | always |
| `seo-aeo` | answer-layer score, top pages | AI answer snapshot: who gets cited | answer-first rewrites, sourced facts | always |
| `seo-agentic` | agent readiness, Lighthouse agentic | | `agentic fix` drafts | always |
| `seo-sxo` | page-type vs SERP match | what page types win each SERP | fix mismatched page types | always |
| `seo-programmatic` | template quality, uniqueness gates | data-driven page opportunities | template improvements | generated pages exist or are planned |
| `seo-indexing` | indexing readiness, crawl cost | | gate parity, sitemaps, rollout plan | 500+ generated URLs, or indexing problems |
| `seo-hreflang` | hreflang/i18n validation | market/language parity | fix hreflang | multilingual or multi-region |
| `seo-local` | GBP, NAP, citations, local schema | local competitors | local pages + schema | physical location or service area |
| `seo-maps` | geo-grid, GBP audit | competitor radius | GBP action list | local business + DataForSEO |
| `seo-ecommerce` | product schema, feeds | marketplace intelligence | product/category fixes | online store |
| `seo-plan` | | strategy for the business type | roadmap in the final report | always |
| `seo-cluster` | | keyword clustering + content architecture | hub/spoke internal links | always |
| `seo-competitor-pages` | | comparison/alternative opportunities | build comparison pages | competitors identified |
| `seo-backlinks` | backlink profile baseline | competitor link gap | outreach list for owner to-do | always (free sources) |
| `seo-flow` | | FLOW Find + Leverage prompts | FLOW Optimize + Win prompts | always |
| `seo-google` | GSC/GA4/CrUX baseline, index status | GSC queries, Keyword Planner | sitemap submission check | Google credentials |
| `seo-drift` | capture baseline | | compare after deploy | live URL |
| `seo-dataforseo` | SERP/rank baseline | volumes, difficulty, competitors, LLM mentions | | DataForSEO extension |
| `seo-firecrawl` | full-site crawl / URL map | crawl competitor sites | | Firecrawl extension |
| `seo-ahrefs` | backlinks/rank baseline | keyword + competitor data | | Ahrefs extension |
| `seo-seranking` | rank/AI visibility baseline | keyword + competitor data | | SE Ranking extension |
| `seo-bing` | Bing index + AI Performance | Bing/Copilot grounding queries | sitemap submission check | Bing Webmaster extension |
| `seo-matomo` | traffic baseline | top pages / referrers | | Matomo extension |
| `seo-profound` | AI visibility baseline | AI citation share vs competitors | | Profound extension |
| `seo-unlighthouse` | site-wide Lighthouse scan | | re-scan after changes | Unlighthouse extension |

Check extension availability once at the start (installed skill present, or
its `*_auth.py --check` succeeds) and record the result in report 1.

---

## Step 1: Audit (check the project, write the report)

1. **Profile.** Run `seo-codebase` Phase 1 (framework, rendering, hosting quotas, routes, real dynamic page counts, existing SEO layer, index gating, crawl cost, internal linking, business context). URL-only mode: detect the business type and site structure the way the `seo` orchestrator does.
2. **Audit.** Run `seo-codebase` Phase 2 on the code. If a live URL exists, also run `/seo audit <url>` and note every place where production differs from the code (stale deploys, CDN caching, blocked resources).
3. **Special checks:**
   - AI answer readiness on the top 5 landing pages (`seo-aeo`).
   - If any generated page type has 500+ URLs: `seo-indexing` in `plan` mode.
   - Search Console / GA4 data if credentials exist (`seo-google`): current clicks, impressions, indexed pages, top queries.
4. **Baseline.** Record the numbers Step 3 will compare against: per-area scores, overall score, indexed vs submitted pages, and GSC clicks/impressions/avg position (last 28 days) if available. If a URL exists, capture a drift baseline (`seo-drift`).

**`01-AUDIT-REPORT.md` must contain:**
1. Summary for the owner (5 lines, plain language): overall score, the biggest problem, the biggest opportunity
2. Project profile (stack, rendering, page counts per route type)
3. Scores table: area → score/100 → one-line reason
4. Critical & high findings (each: evidence `file:line`/URL, impact, fix, how we'd know it worked)
5. Top 5 quick wins
6. Baseline metrics block (used by Step 3)
7. Open questions for the owner (business goal, target locations/audience, main conversion, known competitors)
8. **Skills used**: every Step 1 skill from the coverage table → ran / skipped → reason → output

→ **Stop.** Ask the open questions here; their answers drive Step 2.

## Step 2: Research (keywords + competitors)

Read `01-AUDIT-REPORT.md` and the owner's answers first.

### 2a. Seed topics
From the product itself: what it does, features, pages, locations, audience, the words customers use (site copy, support/sales questions if available, reviews), and existing GSC queries.

### 2b. Keywords
Use the best available source, in this order, and state which you used:
1. **Google Search Console** queries (`seo-google`): real demand already reaching the site, including "striking distance" queries at positions 5-20.
2. **DataForSEO / Keyword Planner / other connected tool** (`seo-dataforseo`, `keyword_planner.py`): volumes, difficulty, SERP features.
3. **Free signals:** Google autocomplete patterns, People Also Ask, related searches, and forum/Reddit phrasing via web search. Volumes here are **tier estimates** (H/M/L), clearly labeled.

For each keyword record: intent (informational / commercial / transactional / navigational / local), the SERP type (what ranks: tools, lists, guides, product pages, local pack, video), SERP features (AI Overview, PAA, snippet), volume or tier (with source), difficulty or estimate, and business value (1-3).

### 2c. Competitors
- **Search competitors ≠ business competitors.** Take the top 5-10 keywords and find which domains actually rank (web search / SERP tools). Include the owner's named competitors too.
- For the 3-5 most frequent domains, fetch their ranking pages and record: page type, depth/coverage, unique assets (tools, data, calculators, reviews), schema, internal-link hubs, freshness, and E-E-A-T signals. With a backlink tool (`seo-backlinks`, DataForSEO), add referring-domain counts; without one, don't guess authority.
- Use `seo-competitor-pages` for comparison/alternative page opportunities, and `seo-sxo` to check that our page type matches what wins each SERP.

### 2d. Gaps and plan
- **Content gaps:** keywords/questions competitors answer and we don't.
- **Quality gaps:** keywords where we have a page but a weaker one (type mismatch, thinner, missing data).
- **Our advantages:** assets we have that competitors lack (original data, tools, local expertise). These are the best targets.
- Cluster the keywords (`seo-cluster`) and map each to **one** page, existing or new. Never map one keyword to two pages (cannibalization).
- **AI answers:** for 10 core questions, note who AI Overviews / ChatGPT / Perplexity cite (sample-based; see `seo-aeo`).

**`02-RESEARCH-REPORT.md` must contain:**
1. Summary (5 lines): the market, who we're up against, where we can win
2. Data sources used, and which numbers are estimates
3. Keyword clusters: cluster → primary keyword → secondaries → intent → volume/tier → target page (✅ existing / 🆕 new)
4. Competitor table: domain → what they rank for → their strengths → their weaknesses → our angle
5. Gap list: content gaps, quality gaps, our advantages
6. AI answer snapshot (who gets cited today)
7. **Prioritized opportunity list** (impact × effort × business value), which becomes Step 3's backlog
8. **Skills used**: every Step 2 skill → ran / skipped → reason → output

**`keyword-map.csv`** columns: `keyword,cluster,intent,volume,volume_source,difficulty,business_value,serp_type,target_url,status`.

→ **Stop.** Show the top 10 opportunities and ask the owner to confirm priorities.

## Step 3: Improve (do the SEO, report what improved)

Read reports 1 and 2.

1. **Build the backlog** in `docs/seo/ACTION-PLAN.md`, merging:
   - audit fixes (Critical/High first: indexing blockers, crawl cost, then quick wins),
   - research opportunities (optimize existing pages for their mapped keywords, then new pages),
   - AEO improvements for top pages, and the indexing plan if relevant.
   Each item: what, why (finding or keyword it serves), files/pages, impact, effort, verification.
2. **Get one approval** for the list (the owner can strike items). Don't edit before this.
3. **Implement** in order, with `seo-codebase` Phase 4 and framework recipes:
   - Technical and indexing fixes (`seo-codebase`, `seo-indexing`).
   - On-page for mapped keywords: title, description, H1, intro that answers the query first, headings covering the cluster's questions, internal links from related pages with descriptive anchors, schema.
   - AEO: answer-first sections, sourced facts, comparison tables, visible dates and authors (`seo-aeo`).
   - **New pages:** build the page/template and structure; write copy only from facts in the codebase, the site, and the owner. Mark any statement needing owner confirmation with `<!-- TODO(owner): verify -->` and list them in the report. Don't publish invented numbers, testimonials, or credentials.
   - Run build/lint/typecheck after each group; tick items off.
4. **Verify** (`seo-codebase` Phase 5): real server HTML, status codes, sitemaps, robots groups, JSON-LD.
5. **Re-score** with the same method as Step 1 so the numbers are comparable.

**`03-IMPROVEMENT-REPORT.md` must contain:**
1. **Summary for the owner** (5-8 lines): what was done, the score change, what to expect and when
2. **Before → after scores**: area → Step 1 score → new score → change → main reason
3. **What we improved**, grouped (Technical & indexing / On-page & keywords / Content & new pages / AI answers / Performance), each change with files changed, the keyword or finding it serves, and how to verify it
4. **Keyword → page map (final)**: which page now targets which cluster
5. **Not done yet**: skipped or deferred items, and why
6. **Owner to-do** (manual steps): submit/resubmit sitemaps in Search Console and Bing, Request Indexing for up to 10 key new/changed pages, Rich Results Test on changed templates, IndexNow key setup, verify `TODO(owner)` facts, Google Business Profile updates, backlink/PR actions from the research
7. **Measurement plan**, **honest about timing**: indexing and crawl changes show in 1-4 weeks; ranking changes typically take several weeks to months; nothing is guaranteed. Checkpoints at 2 weeks, 6 weeks and 3 months, and exactly what to compare (GSC clicks/impressions/position for mapped keywords, indexed pages, AI-report impressions) against the Step 1 baseline
8. **Next cycle**: when to re-run `/seo pipeline` (recommended: monthly audit, quarterly research)
9. **Skills used**: every Step 3 skill → ran / skipped → reason → what it changed

→ **Finish** with a chat summary: the score table, the 5 most important improvements, and the owner to-do list.

## Resuming and re-runs

- Reports carry their date; on re-run, Step 1 compares with the previous `01-AUDIT-REPORT.md` and shows progress since last time.
- If the code changed after Step 2 (`git log` since the report date), re-check affected pages before Step 3.
- Keep old reports as `docs/seo/archive/YYYY-MM-DD-0N-*.md` when overwriting.

## Error Handling

| Scenario | Action |
|---|---|
| No codebase and no URL | Ask for one. Don't guess. |
| No keyword data tools | Use free signals, label every volume as an estimate, and recommend connecting Search Console. |
| Owner skips Step 1/2 questions | Proceed with stated assumptions, listed at the top of the next report. |
| Step 3 requested without reports 1-2 | Offer to run them first; if the owner insists, run a condensed audit, say research was skipped, and don't create new keyword-targeted pages. |
| Build breaks during Step 3 | Revert that change, record it under "Not done yet", continue with independent items. |
