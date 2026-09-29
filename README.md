# Ultimate SEO Skill for Claude Code

**SEO, AEO and GEO for Claude Code: audit any live site, or open any project and let Claude analyze the code, then fix it.** 30 sub-skills and 19 specialist agents cover technical SEO, content quality (E-E-A-T), Schema.org, sitemaps, Core Web Vitals, AI search (GEO), answer engines (AEO), local, e-commerce, international SEO, and getting thousands of generated pages indexed.

Built on [claude-seo](https://github.com/AgriciDaniel/claude-seo) v2.4.0 by AgriciDaniel (MIT), which provides the URL-based audit engine, 19 agents, and Python toolchain. This fork adds a 3-step pipeline and three sub-skills from a real production rollout (a Next.js real-estate site pushing ~5,000 listing pages into Google), plus a September 2026 research pass over primary sources.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

## Start here: `/seo pipeline`

One command, three steps, a report after each. It pauses between steps so you can review.

| Step | Command | What happens | Report |
|---|---|---|---|
| 1. Audit | `/seo pipeline audit` | Checks the project code and live site, scores every area, records baseline numbers | `docs/seo/01-AUDIT-REPORT.md` |
| 2. Research | `/seo pipeline research` | Keywords (Search Console / keyword tools / labeled estimates), who actually ranks, competitor strengths and gaps, who AI answers cite, keyword → page map | `docs/seo/02-RESEARCH-REPORT.md` + `keyword-map.csv` |
| 3. Improve | `/seo pipeline improve` | One approval, then implements the SEO work, verifies it, re-scores, and reports **before → after** with every change and your to-do list | `docs/seo/03-IMPROVEMENT-REPORT.md` |

Just type `/seo pipeline` and it resumes at the next step. **Every skill in this repo is used**: each report ends with a "Skills used" table showing what ran, or why a skill was skipped (for example, local SEO for a business with no location, or a paid extension with no credentials).

## What this fork adds

| Command | What it does | Why it matters |
|---|---|---|
| `/seo pipeline` | The 3-step Audit → Research → Improve flow above, using every sub-skill and extension. | One command from "no SEO" to "done, with proof of what improved". |
| `/seo codebase` | Runs **inside your project**. Profiles the framework, rendering, routes and real dynamic page counts, scores the code, asks for approval, then **implements the fixes** with framework-native recipes (Next.js App/Pages, Astro, Nuxt, SvelteKit, Remix/React Router, Gatsby, WordPress, Hugo/Eleventy/Jekyll, SPAs) and verifies against real server HTML. | URL audits tell you what's wrong; this one changes the code. |
| `/seo indexing` | Gets **hundreds to millions of pages indexed**: one index gate shared by page and sitemap, crawl cost (cached renders, no paid calls on bot hits, image quotas), sitemap chunking, internal-link discovery, staged rollout, Search Console / Bing / IndexNow submission, lifecycle of sold/removed pages, and a fix for every Search Console "not indexed" reason. | Sitemaps get pages *discovered*, not *indexed*. This covers the rest, honestly, including why the Google Indexing API is off-limits for normal pages. |
| `/seo aeo` | **Answer Engine Optimization** across AI Overviews, AI Mode, ChatGPT, Claude, Perplexity and Copilot: eligibility per platform, answer-first structure, question coverage, citable evidence, trust, freshness, and measurement (Search Console generative AI report, Bing AI Performance, prompt panel). | Every recommendation is **evidence-graded** (A platform docs, B peer-reviewed, C industry study) and must fit Google's AI optimization guide. No hacks. |

Plus: the `/seo` orchestrator picks **codebase mode** automatically when you're in a project with no URL, full audits include AEO checks on top landing pages, and sites with 500+ generated URLs get an indexing plan.

## Quick install

### Plugin (Claude Code 1.0.33+)

```bash
/plugin marketplace add Gouravmaurya/ultimate-seo-skill
/plugin install ultimate-seo@gouravmaurya-ultimate-seo
/seo setup
```

### Manual (macOS / Linux)

```bash
git clone --depth 1 https://github.com/Gouravmaurya/ultimate-seo-skill.git
bash ultimate-seo-skill/install.sh
```

### Manual (Windows PowerShell)

```powershell
git clone --depth 1 https://github.com/Gouravmaurya/ultimate-seo-skill.git
powershell -ExecutionPolicy Bypass -File ultimate-seo-skill\install.ps1
```

`/seo setup` creates an isolated Python environment and installs Playwright Chromium for the URL-based tools. `/seo codebase` needs none of that: it works with Claude Code's own file tools.

> If you already have upstream claude-seo installed, uninstall it first. Both install the `seo` skill and the same agent names.

## Get great results: how to use it

1. **Easiest: run `/seo pipeline`** in your project (add your live URL if you have one) and follow the three steps.
2. **Working on your own site's code?** Open the project in Claude Code and run `/seo codebase`. Answer its questions about the business (what you sell, to whom, where, the main conversion). Review `docs/seo/ACTION-PLAN.md`, then tell it which items to implement.
3. **Have thousands of generated pages** (listings, products, locations)? Run `/seo indexing plan`. Fix the gate, crawl cost and internal links **before** submitting sitemaps, then roll out best segments first.
4. **Want AI answers to cite you?** Run `/seo aeo` on your top landing pages, then set up the Search Console and Bing AI reports so you measure instead of guess.
5. **Auditing a live site or a client?** `/seo audit https://example.com` runs the full parallel audit.
6. **Re-run monthly.** Every skill reads its previous output in `docs/seo/` and audits only what changed.

## Principles

- **Evidence over folklore.** Facts are dated and sourced (Google Search Central, OpenAI, Anthropic, Microsoft, peer-reviewed research). Industry statistics are labeled as directional.
- **Honest markup and claims.** No fake reviews, authors or statistics; schema matches visible content; copy matches what the product does.
- **Cheap to crawl = affordable to index.** Scale without burning hosting, API or AI quotas.
- **Stop for approval** before editing code.

---

*The documentation below is from upstream claude-seo and applies to this fork. Where it says `AgriciDaniel/claude-seo`, use `Gouravmaurya/ultimate-seo-skill` to install this version.*

## Quick Start

```bash
# Start Claude Code
claude

# Full site audit: parallel sub-agents produce a prioritized action plan
/seo audit https://example.com

# Deep single-page analysis: on-page elements, content quality, schema
/seo page https://example.com/about

# Schema markup audit: detect, validate, generate
/seo schema https://example.com

# AI search optimization: passage citability + primary-source-aligned recommendations
/seo geo https://example.com

# Generate a sitemap with industry templates
/seo sitemap generate
```

## Commands

![Claude SEO sub-skill ecosystem: 26 modules grouped into 8 categories (audit, content, schema, technical, AI search, local + maps, commerce + intl, extensions) around the central orchestrator](assets/sub-skills.svg)

34 user-invocable `/seo` commands across the orchestrator, its sub-skills, and 9 MCP extensions. Full reference in [docs/COMMANDS.md](docs/COMMANDS.md).

| Command | Description |
|---------|-------------|
| `/seo setup` | Create or refresh the isolated Python runtime and Chromium |
| `/seo doctor` | Check runtime readiness without changing the system |
| `/seo audit <url>` | Full website audit with parallel sub-agent delegation |
| `/seo page <url>` | Deep single-page analysis |
| `/seo technical <url>` | Technical SEO audit across 9 categories |
| `/seo content <url>` | E-E-A-T and content quality analysis |
| `/seo content-brief <topic>` | Detailed content brief: target keywords, outline, internal links |
| `/seo schema <url>` | Detect, validate, and generate Schema.org markup |
| `/seo geo <url>` | AI Overviews / Generative Engine Optimization |
| `/seo agentic <url>` | Agent readiness: Lighthouse Agentic Browsing X/N, AI agent access, llms.txt, Markdown, WebMCP |
| `/seo sitemap <url \| generate>` | Analyze or generate XML sitemaps |
| `/seo images <url>` | Image optimization analysis |
| `/seo plan <type>` | Strategic SEO planning (saas, local, ecommerce, publisher, agency) |
| `/seo programmatic <url>` | Programmatic SEO analysis and planning |
| `/seo competitor-pages <url>` | Competitor comparison page generation |
| `/seo local <url>` | Local SEO analysis (GBP, citations, reviews, map pack) |
| `/seo maps [command]` | Maps intelligence (geo-grid, GBP audit, reviews, competitors) |
| `/seo hreflang <url>` | Hreflang / i18n SEO audit and generation |
| `/seo google [command]` | Google SEO APIs (GSC, PageSpeed, CrUX, Indexing, GA4, PDF reports) |
| `/seo backlinks <url>` | Backlink profile analysis (Moz, Bing, Common Crawl) |
| `/seo cluster <keyword>` | SERP-based semantic clustering |
| `/seo sxo <url>` | Search Experience Optimization (page-type, user stories, personas) |
| `/seo drift baseline \| compare \| history <url>` | SEO drift monitoring with SQLite snapshots |
| `/seo ecommerce <url>` | E-commerce SEO and marketplace intelligence |
| `/seo flow [stage]` | FLOW framework prompts (CC BY 4.0, evidence-led) |
| `/seo firecrawl [command] <url>` | Full-site crawling (extension) |
| `/seo dataforseo [command]` | Live SEO data (extension) |
| `/seo image-gen [use-case]` | AI image generation for SEO assets (extension) |
| `/seo ahrefs [command] <url>` | Backlinks, organic keywords, and content data via the official Ahrefs MCP (extension) |
| `/seo seranking [command]` | AI Share-of-Voice across ChatGPT, Gemini, Perplexity, AI Overviews, AI Mode (extension) |
| `/seo profound [command]` | LLM citation tracking with time-series data (extension) |
| `/seo bing [command] <url>` | Bing Webmaster Tools + IndexNow URL submission (extension) |
| `/seo matomo [command] [args]` | Matomo Reporting API: GA4 alternative or complement (extension) |
| `/seo unlighthouse <url>` | Multi-page Lighthouse runner, runs locally (extension) |

## Features

### What Core Web Vitals does Claude SEO check?

Claude SEO measures the current three Core Web Vitals: **LCP** (Largest Contentful Paint, target under 2.5s), **INP** (Interaction to Next Paint, target under 200ms), and **CLS** (Cumulative Layout Shift, target under 0.1). [INP replaced FID](https://web.dev/articles/inp) on March 12, 2024; FID was removed from Chrome's field-data tools (CrUX API, PageSpeed Insights) on September 9, 2024 (Lighthouse is a lab tool and never reported FID), and Claude SEO never references FID. Field data comes from the Chrome User Experience Report (CrUX) when available; lab data falls back to Lighthouse via PageSpeed Insights. LCP can be decomposed into subparts (TTFB, load delay, load duration, render delay) via the `/seo google` CrUX integration to localize bottlenecks. Mobile and desktop are measured separately. CrUX History (25-week trend) is included in the Tier 0 free credential set.

### How does Claude SEO assess E-E-A-T?

E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) is evaluated against the Search Quality Rater Guidelines, last updated September 2025 with YMYL expanded to include political and social topics. Experience signals: original research, case studies, first-hand photos. Expertise: author credentials and topical depth. Authoritativeness: external citations and brand mentions. Trustworthiness, the most heavily weighted of the four: contact info, secure HTTPS, transparent corrections, date stamps. Before scoring sub-factors, Claude SEO applies Google's own Who / How / Why heuristic from the [helpful-content guide](https://developers.google.com/search/docs/fundamentals/creating-helpful-content). Generative AI content is fine if it meets Search Essentials; it crosses into spam when used to scale low-value pages, which `seo-content humanize` and `seo-content verify` are designed to detect.

### What Schema.org types does Claude SEO support?

JSON-LD is the preferred format (Google's stated preference). Claude SEO detects, validates, and generates the active Schema.org types documented in [skills/seo/references/schema-types.md](skills/seo/references/schema-types.md), including organization, article, product, local, event, job, course, software/application, service, Q&A, and video patterns. FAQPage: Google stopped showing FAQ rich results for all sites on May 7, 2026; it has no Google rich-result benefit. Keep it only for non-Google or internal semantics if needed. Deprecated and never recommended: HowTo (rich results removed September 2023), SpecialAnnouncement (July 2025), ClaimReview, VehicleListing, EstimatedSalary, LearningVideo, CourseInfo carousel (all retired June 2025). Replacement guidance: [skills/seo-schema/references/deprecated-types-2024-2026.md](skills/seo-schema/references/deprecated-types-2024-2026.md).

### How does Claude SEO optimize for AI search?

Aligned with [Google's AI Optimization Guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide), which says that "optimizing for generative AI search is optimizing for the search experience, and thus still SEO." AI Overviews and AI Mode are grounded in the same ranking systems as classic Search; pages must be indexed, eligible for snippet display, and included in the Search Console generative AI setting (the default) to appear in any AI feature. Claude SEO scores passage citability (self-contained answer blocks; ~130-170 words is a third-party heuristic, not a Google rule), question-based heading hierarchy, attribution density, structured data coverage, and entity presence across Wikipedia, Reddit, YouTube, and LinkedIn. The `seo-geo` skill includes evidence-based reframes of three popular myths: llms.txt is not currently a citation lever ([primary-source evidence](skills/seo-geo/references/llmstxt-evidence.md)), content chunking is not required, and AI-specific keyword rewriting is unnecessary because synonym understanding is sufficient.

### Which Google SEO APIs does Claude SEO integrate with?

A 4-tier credential system lets you start with zero keys and add data as needed. Every tier delivers real value at its level:

| Tier | Credentials | APIs Unlocked |
|------|------|------|
| 0 | API key | PageSpeed Insights, CrUX, CrUX History (25-week trends) |
| 1 | + OAuth or Service Account | + Search Console (queries, URL Inspection, sitemap status), Indexing API |
| 2 | + GA4 property config | + GA4 organic traffic, top landing pages, device / country breakdown |
| 3 | + Ads developer token | + Keyword Planner search volume and competition data |

PDF reports are generated via [WeasyPrint](https://weasyprint.org/) (A4 layout) with matplotlib charts at 200 DPI. Run `/seo google setup` for the credential wizard. All credentials live under `~/.config/claude-seo/` with `0o600` permissions; nothing is checked into the repo.

### How does Claude SEO handle local SEO?

Three layers. **Google Business Profile signals**: categories, hours, photos, posts, products, attributes. **NAP consistency** across citations: name, address, phone matched against major directories with deviation flagging. **Review intelligence**: rating trends, sentiment, response coverage. For multi-location businesses, Claude SEO enforces a 30-page warning threshold and a 50-page hard stop to prevent doorway-page violations (configurable). The `/seo maps` workflow adds geo-grid rank tracking, GBP profile auditing, and competitor radius mapping. Local schema generation covers `LocalBusiness` with all required and recommended properties (geo coordinates, opening hours, areaServed). Phase F (v2) added a GBP deprecation linter that detects retired chat-field references and `.business.site` URLs.

## Compared to manual / agency / commercial tools

| | Manual audit | Agency engagement | Commercial SEO audit tool | **Claude SEO** |
|---|---|---|---|---|
| **Time per audit** | 4-8 hrs senior SEO time | 1-3 weeks turnaround | 10-45 min crawl + report | **10-15 min** |
| **Cost** | High (billable hours) | $2k-$15k+ project | $99-$999/mo subscription | **Free skill + Claude Code subscription** |
| **Repeatable** | Inconsistent across analysts | Inconsistent across engagements | Yes | **Yes, deterministic + scriptable** |
| **Output format** | Wall-of-findings PDF | Branded slide deck | Web dashboard, CSV exports | **Markdown + PDF + JSON, local files** |
| **Custom benchmarks** | Manual per analyst | Agency-specific frameworks | Vendor-fixed | **Edit local SKILL.md** |
| **Data leaves machine?** | No (your spreadsheet) | Yes (sent to agency) | Yes (uploaded to vendor) | **No, fully local by default** |
| **Lock-in** | None | High | High (data-exit friction) | **None. MIT, your files.** |
| **AI search awareness** | Depends on analyst | Depends on agency seniority | Lagging (typically 6-12 mo behind) | **Google AI Optimization Guide (May 2026), Sept 2025 QRG, INP-not-FID, GEO/AEO=SEO reframe, llms.txt evidence-based posture** |
| **Falsifiability per finding** | No | No | No | **Yes. Every recommendation carries a "how would we know this failed?" check + leading indicator** |

> Cost benchmarks: manual audit assumes a senior SEO consultant at typical agency billable rates; agency engagement based on common discovery/audit deliverable scopes; commercial-tool subscriptions reflect published mid-tier pricing across the SEO audit category (Ahrefs, Semrush, Sitebulb, Screaming Frog). Your numbers may differ.

## Use cases

**SEO agency lead running 10 client sites.** Replaces the quarterly "deep audit" ritual with a weekly Monday-morning `/seo audit` run per site. Time to deliver a client health-score email drops from 4 hours to 12 minutes; coverage goes from quarterly to weekly without billing more hours. The drift baseline catches regressions between audits so the client conversation moves from "look at this snapshot" to "here is what changed this week."

**In-house SEO lead at a 50-person SaaS company.** Runs `/seo audit` 24 hours before each quarterly business review. Catches the items the platform UI buries (broken canonical chains on programmatic pages, schema deprecation after Google's June 2025 retirement wave, AI-citability gaps that erode SERP-to-AI-Overview pickup, expired-domain heritage on acquired blog assets) before the CMO asks why organic traffic is down in front of the board.

**Freelance SEO consultant onboarding a new client.** Runs `/seo audit` on the discovery call. Anchors the engagement scope with a real 0-100 score, 3 prioritized critical findings, and a falsifiability check on each recommendation, instead of a vague "I'll take a look and get back to you." Closes more retainers because the proof of value happens during the call, not after the proposal.

## Sample Output

Claude SEO writes real markdown reports as its primary deliverable. Below is the first ~50 lines of a `/seo schema https://rankenstein.pro/about` audit verbatim. The actual structure, headers, and grading format the plugin produces follows.

<details>
<summary><code>SCHEMA-REPORT.md</code>: first 50 lines of a real audit</summary>

```markdown
# Schema Markup Report: rankenstein.pro/about

**URL:** https://rankenstein.pro/about
**Date:** 2026-02-09
**Format Detected:** JSON-LD (3 blocks) | No Microdata | No RDFa

---

## Summary

| Metric | Value |
|--------|-------|
| **JSON-LD Blocks** | 3 |
| **Schema Types** | Organization, WebSite, SoftwareApplication |
| **Critical Issues** | 2 |
| **Warnings** | 5 |
| **Passed Checks** | 18 |
| **Overall Grade** | B+ (solid foundation, actionable gaps) |

---

## Existing Schema Validation

### 1. Organization (`@id: #organization`)

| Property | Value | Status | Notes |
|----------|-------|--------|-------|
| `@context` | https://schema.org | Valid | |
| `@type` | Organization | Valid | Active type |
| `@id` | https://rankenstein.pro#organization | Good | Enables cross-referencing |
| `name` | Rankenstein | Valid | |
| `description` | Present, 200+ chars | Good | Descriptive and keyword-rich |
| `url` | https://rankenstein.pro | Valid | Absolute URL |
| `logo` | ImageObject with @id, url, width, height, caption | Excellent | Well-structured |
| `foundingDate` | "2024" | Imprecise | Year-only accepted but ISO 8601 preferred |
| `areaServed` | "Worldwide" | Text | Works but `GeoShape` is more semantic |
| `contactPoint` | email + contactType | Valid | Consider adding `telephone` |
| `founder` | 1 Person (Daniel Agrici) | Incomplete | Page describes two co-founders; second missing |
| `sameAs` | 5 social profiles | Good | GitHub, X, LinkedIn, YouTube, Reddit |
| `knowsAbout` | 6 topics | Good | Relevant topical signals |

**Critical Issue:** The `founder` property only includes Daniel Agrici. Benjamin Samar (Co-Founder & Technical Director) is displayed on the page but absent from the schema. This creates a content-schema mismatch that can confuse search engines.
```

</details>

Other audit outputs follow the same shape: `FULL-AUDIT-REPORT.md` (umbrella audit), `GEO-ANALYSIS.md` (AI-search readiness), `LOCAL-SEO-ANALYSIS.md` (GBP and citations), and a production PDF via WeasyPrint + matplotlib (cover, TOC, executive summary, data sections, recommendations, methodology, roughly 32 A4 pages for a full site audit).

## Architecture

![Claude SEO audit signal flow: /seo audit enters the orchestrator, fans out to 26 sub-skills and up to 17 parallel audit agents, and converges through the scoring engine into a prioritized report](assets/signal-flow.svg)

The plugin follows the [Agent Skills standard](https://docs.claude.com/en/docs/claude-code/skills) with a 3-layer architecture (directive, orchestration, execution). Skills and agents are auto-discovered from `skills/seo-*/` and `agents/seo-*.md`. The orchestrator (`skills/seo/SKILL.md`) handles industry detection (SaaS, local, ecommerce, publisher, agency), parallel sub-agent dispatch up to 17 simultaneously, and synthesis through the [10-principle framework](#methodology) before emitting the action plan. Full architecture: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Methodology

![Claude SEO 10-principle methodology: PERCEIVE, ANALYZE, VALIDATE, and ACT phases with 10 principles arranged by quadrant](assets/framework.svg)

Every audit walks 10 principles grouped into four phases. Each emitted recommendation carries four fields: the first-principle observation it rests on, its dependency relationship to other recommendations, a "how would we know this failed?" check, and a leading indicator to monitor.

| Phase | Principles | What it does |
|---|---|---|
| **PERCEIVE** | OBSERVE (external) · OBSERVE (internal) · LISTEN | Collect raw signals; audit your own assumptions; read what the SERP, the brand voice, and the community actually say |
| **ANALYZE** | THINK · CONNECT (lateral) · CONNECT (system) | Reduce to first principles; find non-obvious cross-skill links; sequence into a dependency graph |
| **VALIDATE** | FEEL · ACCEPT | Pressure-test against UX, brand voice, operator capacity; surface falsifiability |
| **ACT** | CREATE · GROW | Ship the artifact; set the feedback loop for the next audit |

Full methodology: [skills/seo/references/thinking-framework.md](skills/seo/references/thinking-framework.md).

## What's New in v2

v2.0.0 is the largest release in the plugin's history. Six build phases, all shipped:

- **Phase A: Headless rendering everywhere.** Shared `scripts/render_page.py` with Playwright Chromium plus [trafilatura](https://github.com/adbar/trafilatura) and [htmldate](https://github.com/adbar/htmldate). Every audit subagent gets SPA-aware fetching via `--render auto` (auto-detected on Next.js, React, Vue, Nuxt, Astro islands). Closes the SPA limitation that capped v1.x.
- **Phase B: QRG-aligned content quality gates.** Filler detector and AI-pattern humanizer keyed to QRG §4.6.5 and §4.6.6, claim-verification scanner, expired-domain heritage check via WHOIS, primary-source Google updates changelog.
- **Phase C: Technical and CWV depth.** LCP subparts via CrUX (TTFB, load delay, load duration, render delay), Speculation Rules and bfcache detection, IndexNow submitter for Bing / Yandex / Seznam / Naver, Unlighthouse multi-page Lighthouse wrapper.
- **Phase D: Schema completeness.** Four explicit generators (Reservation, OrderAction, DiscussionForumPosting, ProfilePage), e-commerce schema validator (`hasMerchantReturnPolicy`, `shippingDetails`, `MemberProgram`, EU `energyEfficiencyClass`, ProductGroup variants), dual validator (Rich Results Test plus Schema Markup Validator).
- **Phase E: AI search reframing and 5 new MCP extensions.** Ahrefs, SE Ranking (AI Share-of-Voice), Profound (LLM citation tracker), Bing Webmaster plus IndexNow, Unlighthouse. Plus the parasite-SEO risk scanner per Google's November 2024 [site reputation abuse policy](https://developers.google.com/search/blog/2024/11/site-reputation-abuse-update).
- **Phase F: Local, international, and privacy polish.** Google Business Profile deprecation linter (chat field and `.business.site` URLs, with Q&A treated as category/region-limited), DMA consent-mode-v2 click-through diagnostic, machine-translation QA flag per January 2025 QRG.

Test coverage grew from 39 (v1.9.9) to 410 across the v2 line; the url_safety suite alone runs 91 SSRF and DNS-rebinding bypass cases, closing the obfuscated-IPv4, FQDN-trailing-dot, and redirect-rebinding bypass classes. Full migration notes and breaking changes: [docs/MIGRATION-v1-to-v2.md](docs/MIGRATION-v1-to-v2.md).

### Since v2.0.0

- **v2.1.0 (May 2026): currency refresh.** May 2026 core update, Google I/O 2026 (custom version of Gemini 2.5 powers AI Mode), FAQ rich results retired 2026-05-07 (QAPage remains the type for genuine Q&A pages, FAQ markup itself just no longer yields rich results).
- **v2.2.0 (June 2026): security + portability.** Installer credential-injection fix, SSRF authority-confusion bypass closed, Google API keys moved to the `X-Goog-Api-Key` header, secret-scan CI gate, Windows/macOS fixes; suite at 326.
- **v2.2.1 (June 2026): Google-currency reconfirmation + full command audit.** Lighthouse 13.4.0 with the new Agentic Browsing category, Google Search ignores llms.txt, an internally-reweighted E-E-A-T scorecard (Trust highest, per Google's 'trust is most important'); every `/seo` command and subcommand audited and COMMANDS.md brought to 100% coverage.
- **v2.2.2 (July 2026): full-review maintenance.** Corrected GBP Q&A handling, AI Mode model naming, image-model IDs, hook input behavior, and added a strict reference-graph consistency gate.
- **v2.2.3 (July 2026): prompt-hygiene alignment.** Normalized emphasis and punctuation across the prompt surface without changing behavior, routing, or output contracts.
- **v2.2.4 (July 2026): community maintenance.** Added the managed cross-platform runtime and safe sitemap discovery, repaired GSC pagination and totals, replaced removed Bing endpoints, fixed extension and Windows portability gaps, and reconciled every open issue and pull request.
- **v2.2.5 (August 2026): reliability and Google-currency hardening.** Fixed manual-install data packaging and lxml runtime imports, hardened JSON-LD graphs and rendered-page accessibility analysis, enforced managed-runtime command references, and refreshed Google Search and Lighthouse guidance through August 25.
- **v2.2.6 (September 2026): security and hosted-install patch.** Closed a cache-path traversal and a WHOIS referral SSRF, refused the RFC 6598 range, moved the launcher to `scripts/` so the claude.ai-hosted marketplace accepts the plugin, raised dependency floors past PYSEC-2026-3940, locked the cost ledgers, and added Windows, macOS, and pip-audit CI.
- **v2.3.0 (September 2026): Google currency and community fixes.** Refreshed Google guidance through 2026-09-10 (EEA site-reputation enforcement, regional aggregator and supplier units, AI Mode travel, CrUX August), added the `CLAUDE_SEO_LOCAL_TARGETS` allowlist and proxy validation, hardened the JSON-LD hook and Unicode watermark stripping, corrected AI crawler guidance, unified `fetch_page --json`, fixed Unlighthouse flags, raised audit agent turn budgets, and landed 12 more community PRs.
- **v2.3.1 (September 2026): agents on Opus and community follow-ups.** Five judgment-heavy agents run on Opus, Keywords Everywhere joins as a free backlinks fallback, setup failures show the failing stage's output, the remaining extension installers write `~/.claude.json` atomically, and the v2.3.0 templated-metadata script is registered with the launcher.
- **v2.4.0 (September 2026): agent readiness and audit fixes.** New `/seo agentic` scores Lighthouse's Agentic Browsing category exactly as Lighthouse counts it and audits AI agent access, llms.txt, Markdown delivery, `ai-catalog.json` and WebMCP; extension installers keep secrets off the command line and never wipe a malformed config; fixes #312, #314 and #317; facts re-verified against primary sources.

## Limitations

Two real boundaries worth being upfront about.

**Heavy client-side hydration timing.** Phase A's headless renderer handles most SPAs out of the box (`--render auto` detects empty `<div id="root">` shells and switches to Playwright). Edge cases that still produce noisy findings: pages with hydration tied to scroll position past the fold, pages that fetch critical content after user interaction (modal opens, tab clicks), pages with race-condition-prone third-party widget mounts. For these, manually triggering the `seo-visual` subagent and comparing its Playwright snapshot to the raw-HTML subagents' findings is the recommended workflow.

**Local-only without enrichment.** The free tier makes no third-party API calls by default (audits still fetch the target URLs you point them at). Adding Google API credentials (Tier 0 through 3) unlocks real field data and live indexation status; without them, Core Web Vitals are lab estimates only and indexation is inferred from page-level signals. Adding MCP extensions (Ahrefs, DataForSEO, SE Ranking, Profound) similarly unlocks competitive and AI-citation data but requires their respective accounts.

## Requirements

**Agent models.** Five judgment-heavy agents (`seo-content`, `seo-geo`, `seo-sxo`,
`seo-cluster`, `seo-drift`) run on Opus since v2.3.1; the other thirteen run on
Sonnet. A full `/seo audit` therefore costs more than it did on v2.3.0. To change
a model, edit the `model:` line in the agent's frontmatter under `agents/`.


- Python 3.10+
- Claude Code CLI
- Optional: Playwright Chromium — install.sh offers to install it (you can skip the prompt); needed only for SPA rendering and screenshots
- Optional: Google API credentials for enriched CWV / GSC / GA4 data (see `/seo google setup`)

## Uninstall

```bash
git clone --depth 1 https://github.com/AgriciDaniel/claude-seo.git
bash claude-seo/uninstall.sh
```

<details>
<summary>One-liner (curl)</summary>

```bash
curl -fsSL https://raw.githubusercontent.com/AgriciDaniel/claude-seo/main/uninstall.sh | bash
```

</details>

## Extensions

Optional MCP servers add live data to the audit pipeline. Claude SEO ships extensions for 9 servers; the plugin core works without any of them.

### DataForSEO

Live SERP data, keyword research, backlinks, on-page analysis, content analysis, business listings, AI visibility checks, and LLM mention tracking. 23 data commands across 9 API modules.

```bash
./extensions/dataforseo/install.sh   # requires DataForSEO account
/seo dataforseo serp best coffee shops
/seo dataforseo ai-mentions your brand
```

Full DataForSEO docs: [extensions/dataforseo/README.md](extensions/dataforseo/README.md).

### Firecrawl

Full-site crawling and URL discovery via the [Firecrawl](https://www.firecrawl.dev/) MCP server.

```bash
./extensions/firecrawl/install.sh
/seo firecrawl crawl https://example.com
```

Full Firecrawl docs: [extensions/firecrawl/README.md](extensions/firecrawl/README.md).

### Banana: AI image generation

SEO image generation (OG previews, blog heroes, product photos, infographics) via the [Claude Banana](https://github.com/AgriciDaniel/banana-claude) Creative Director pipeline.

```bash
./extensions/banana/install.sh
/seo image-gen og "Professional SaaS dashboard"
```

Full Banana docs: [extensions/banana/README.md](extensions/banana/README.md).

### Ahrefs, SE Ranking, Profound, Bing Webmaster, Unlighthouse (new in v2)

Five extensions added in Phase E:

- **Ahrefs:** official `@ahrefs/mcp` server with backlink and organic data
- **SE Ranking:** AI Share-of-Voice across ChatGPT, Gemini, Perplexity, AI Overviews, AI Mode
- **Profound:** LLM citation tracker with time-series data
- **Bing Webmaster:** Bing Webmaster Tools plus IndexNow unified
- **Unlighthouse:** MIT-licensed multi-page Lighthouse runner

### Matomo (analytics without Google)

Self-hosted (or Matomo Cloud) Reporting API as a GA4 alternative or
complement. Useful for privacy-first analytics, regulated industries,
and teams who own their analytics data. Adds organic traffic trend,
landing pages, device / country breakdowns, and referrer analysis.

```bash
./extensions/matomo/install.sh   # requires Matomo instance URL + API token_auth
/seo matomo organic
/seo matomo referrers
```

Full Matomo docs: [extensions/matomo/docs/MATOMO-SETUP.md](extensions/matomo/docs/MATOMO-SETUP.md).
The audit orchestrator spawns the `seo-matomo` agent automatically when
credentials are present.

Setup walkthroughs live under `extensions/<name>/docs/`; integration notes: [docs/MCP-INTEGRATION.md](docs/MCP-INTEGRATION.md).

## Ecosystem

Claude SEO is part of a family of Claude Code skills that interoperate cleanly:

| Skill | What it does | How it connects |
|-------|-------------|-----------------|
| [Claude SEO](https://github.com/AgriciDaniel/claude-seo) | SEO analysis, audits, schema, GEO | Core. Analyzes sites and generates action plans. |
| [Claude Blog](https://github.com/AgriciDaniel/claude-blog) | Blog writing, optimization, scoring | Companion. Writes content optimized by SEO findings. |
| [Claude Banana](https://github.com/AgriciDaniel/banana-claude) | AI image generation via Gemini | Shared. Generates images for SEO assets and blog posts. |
| [Codex SEO](https://github.com/AgriciDaniel/codex-seo) | Codex-first SEO skill suite | Port. Same SEO system adapted for Codex skills, TOML agents, deterministic runners. |
| [AI Marketing Claude](https://github.com/zubair-trabzada/ai-marketing-claude) | Copywriting, emails, social, ads, funnels, CRO | Community. Post-audit marketing action from SEO findings. |
| [FLOW](https://github.com/AgriciDaniel/flow) | Evidence-led SEO framework (41 AI prompts, CC BY 4.0) | Knowledge base. Powers `seo-flow` prompts. |

**Workflow example:**

1. `/seo audit https://example.com`: identify content gaps and technical issues
2. `/seo backlinks https://example.com`: analyze link profile and competitor gaps
3. `/seo geo https://example.com/blog/post`: score AI-citation readiness
4. `/blog write "target keyword"`: create SEO-optimized blog post (Claude Blog)
5. `/seo image-gen hero "blog topic"`: generate hero image (Banana extension)

## Documentation

- [Installation Guide](docs/INSTALLATION.md)
- [Commands Reference](docs/COMMANDS.md): every `/seo` command in depth
- [Architecture](docs/ARCHITECTURE.md): 3-layer design, auto-discovery, parallel dispatch
- [Migration v1 → v2](docs/MIGRATION-v1-to-v2.md): breaking changes, six phases of work
- [MCP Integration](docs/MCP-INTEGRATION.md): integration notes; extension setup lives under `extensions/<name>/docs/`
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Contributors](CONTRIBUTORS.md): community credits

## FAQ

### What is Claude SEO?

Claude SEO is an open-source SEO analysis plugin for Claude Code. It runs 26 sub-skills and 19 specialist agents in parallel across technical SEO, content quality, Schema.org markup, AI search optimization, local SEO, e-commerce, and international SEO. Audits produce a prioritized action plan where each recommendation carries the first-principle observation it rests on, its dependency relationship to other recommendations, a "how would we know this failed?" check, and a leading indicator. The plugin is MIT-licensed, ships zero proprietary tracking, and works without third-party API enrichment; audits still contact the target URLs you analyze. Aligned with [Google's AI Optimization Guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide) and the September 2025 Quality Rater Guidelines.

### How is Claude SEO different from Screaming Frog or Ahrefs Site Audit?

Different surface area, different tradeoffs. **Screaming Frog** crawls deeper and faster at the link-graph level; it is purpose-built as a crawler and Claude SEO does not attempt to replace it. **Ahrefs Site Audit** brings a proprietary backlink index and link intelligence; Claude SEO integrates with Ahrefs via its MCP extension rather than competing. Where Claude SEO leads: conversational LLM-native workflow, recommendation falsifiability (every finding carries an explicit failure-mode check), open-source MIT licensing with zero per-domain pricing, AI search optimization aligned with Google's primary-source guidance, and primary-source schema-deprecation tracking. Use Screaming Frog or Ahrefs for what they are best at; use Claude SEO when you want LLM-driven synthesis, conversational iteration, and AI-search-first audits in the same environment as your other Claude Code workflows.

### Does Claude SEO work on single-page applications (Next.js, React, Vue)?

Yes. Phase A of v2 shipped a shared headless renderer (`scripts/render_page.py`) backed by Playwright Chromium. Audit subagents call `render_page.py --mode auto`, which auto-detects SPA hallmarks (empty `<div id="root">` shells, single bundle script, hydration markers) and switches to a rendered fetch. The lower-level `scripts/fetch_page.py` wrapper supports `--render auto` as an opt-in wrapper mode; its default is `--render never` for raw HTTP. Use `render_page.py --mode always` or `fetch_page.py --render always` to force rendering. Content extraction uses [trafilatura](https://github.com/adbar/trafilatura) for boilerplate removal. Publication dates come from [htmldate](https://github.com/adbar/htmldate). Known nuance: pages with scroll-bound hydration or post-interaction content fetches still produce noisy findings; see the [Limitations](#limitations) section for the recommended `seo-visual` cross-check workflow on those edge cases.

### What Google APIs does Claude SEO use, and are they required?

None are required. Claude SEO is fully functional with zero API keys. A 4-tier credential system lets you upgrade gradually: Tier 0 (API key only) unlocks PageSpeed Insights, CrUX, and CrUX History (25-week trend data). Tier 1 (+ OAuth or service account) adds Search Console with queries, URL Inspection, sitemap status, and the Indexing API for eligible JobPosting pages or BroadcastEvent in VideoObject pages; the API does not guarantee indexing. Tier 2 (+ GA4 property config) adds organic traffic, top landing pages, and device / country breakdowns. Tier 3 (+ Ads developer token) adds Keyword Planner search volume and competition data. The credential setup wizard runs via `/seo google setup`. All credentials live under `~/.config/claude-seo/` with `0o600` file permissions; nothing is checked into the repo and nothing is transmitted beyond Google's own endpoints.

### Is Claude SEO free?

Yes. MIT licensed, fully open source, no per-domain pricing, no telemetry, no API quotas imposed by the plugin itself. The core plugin and all 26 sub-skills work without any paid service. Some optional MCP extensions wrap paid services (DataForSEO, Ahrefs, Profound, SE Ranking) where you bring your own account credentials; their use is opt-in and the plugin works fully without them. Google APIs (PageSpeed Insights, Search Console, Indexing, GA4) are free from Google with normal account quota limits and require your own credentials. If you want commercial support or enterprise features beyond the open-source plugin, that is not part of this project.

### How is Claude SEO different from regular SEO tools when it comes to AI search?

Most SEO tools treat AI search as a separate optimization discipline. Claude SEO follows [Google's own position](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide) that optimizing for generative AI search is "still SEO". AI Overviews and AI Mode are grounded in the same ranking systems as classic Search; the eligibility floor is normal indexation. Claude SEO scores passage citability (self-contained answer blocks), question-based heading hierarchy, attribution density, and entity presence across Wikipedia, Reddit, YouTube, and LinkedIn. It explicitly rejects three influencer myths: llms.txt as a citation lever, content chunking for AI, and AI-specific keyword rewriting. For commerce sites, Claude SEO audits the IPTC `TrainedAlgorithmicMedia` requirement on AI-generated product images per Google Merchant Center policy.

## Community Contributors

v1.9.0 includes contributions from the [AI Marketing Hub](https://www.skool.com/ai-marketing-hub) Pro Hub Challenge:

| Contributor | Contribution |
|------------|-------------|
| **Lutfiya Miller** (Winner) | Semantic Cluster Engine → `seo-cluster` |
| **Florian Schmitz** | SXO Skill → `seo-sxo` |
| **Dan Colta** | SEO Drift Monitor → `seo-drift` |
| **Chris Muller** | Multi-lingual SEO → `seo-hreflang` enhancements |
| **Matej Marjanovic** | E-commerce + DataForSEO Cost Config → `seo-ecommerce` + cost guardrails |

See [CONTRIBUTORS.md](CONTRIBUTORS.md) for full details and original repo links.

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting PRs and include the tests or checks you ran in the PR description.

---

## Author

Built by **[Agrici Daniel](https://agricidaniel.com/about)**, AI Workflow Architect. Single maintainer, open to community contributions via the [Pro Skool community](https://www.skool.com/ai-marketing-hub-pro). Background in marketing automation, AI-assisted content workflows, and open-source tooling for Claude Code.

- [Blog](https://agricidaniel.com/blog): deep dives on AI marketing automation
- [AI Marketing Hub (free)](https://www.skool.com/ai-marketing-hub): open community
- [AI Marketing Hub Pro](https://www.skool.com/ai-marketing-hub-pro): Pro community, early access to this skill
- [YouTube](https://www.youtube.com/@AgriciDaniel): tutorials and demos
- [GitHub](https://github.com/AgriciDaniel): all open-source tools
