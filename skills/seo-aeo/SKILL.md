---
name: seo-aeo
description: >
  Answer Engine Optimization: make pages the source AI answers and direct
  answers quote, across Google AI Overviews and AI Mode, ChatGPT search,
  Claude, Perplexity, and Copilot. Audits the answer layer of pages
  (answer-first structure, question coverage, citable facts, sources,
  freshness, entity clarity, snippet controls, per-platform access) with
  evidence-graded recommendations, and measures AI visibility with Search
  Console and Bing AI reports. Use when user says "AEO", "answer engine",
  "get cited by ChatGPT", "AI Overviews citations", "AI Mode", "featured
  snippet", "people also ask", or "zero-click".
user-invocable: true
argument-hint: "[url or path]"
license: MIT
metadata:
  author: Gouravmaurya
  version: "2.6.0"
  category: seo
---

# AEO: Answer Engine Optimization

**Scope.** `seo-geo` covers AI crawler access, brand/entity signals, and
llms.txt. `seo-agentic` covers agent readiness. **This skill covers the
answer layer**: whether a page gives an answer an engine can find, trust,
quote and attribute, and how to measure that. It works on a live URL or on
source files (with `seo-codebase`).

## The ground rule: SEO first, no hacks

Google's AI optimization guide (updated 2026-07-10) says optimizing for
generative AI search "is still SEO". It explicitly rejects:
- `llms.txt` or other special AI files/markup for Google;
- "chunking" content into tiny pieces, or rewriting it "for AI";
- separate pages for every query variation (**scaled content abuse**);
- chasing every long-tail variant, exact-match keywords, or inauthentic mentions;
- "AEO/GEO hacks" and tools claiming access to internal Google metrics.

It recommends: unique, non-commodity, people-first content; clear sections
and headings; supporting images/video; crawlable, indexed, snippet-eligible
pages; semantic HTML; good page experience; Merchant Center/Business Profile
where relevant; and the Search Console generative AI report for measurement.

**Every recommendation this skill makes must be compatible with that guide.**
When a third-party tactic conflicts with it, drop the tactic or report the
conflict.

## Evidence grades

Tag each recommendation with its grade (see `references/evidence.md`):

| Grade | Source type | How to use it |
|---|---|---|
| **A** | Primary platform documentation (Google, OpenAI, Anthropic, Microsoft) | Rules. |
| **B** | Peer-reviewed research (e.g. GEO, KDD 2024) | Strong guidance. State the conditions tested. |
| **C** | Third-party industry studies (Ahrefs, SE Ranking, Semrush, etc.) | Directional only. Always cite source and date; never present as a rule. |
| **D** | Anecdote / vendor marketing | Don't use as a basis for recommendations. |

Never quote a statistic without its source and date.

## Audit: the answer layer

Score each page 0-100 with these weights.

### 1. Eligibility (gate; 20%) [A]
Before anything else, check that the page can be used as an answer at all:
- Indexed, 200, canonical, in server HTML (many AI crawlers don't run JS).
- Snippets allowed: no `nosnippet`, no tiny `max-snippet`, key text not inside `data-nosnippet`. These govern AI Overviews and AI Mode too.
- Search bots allowed per platform: `Googlebot` (AI Overviews / AI Mode), `OAI-SearchBot` (ChatGPT search), `Claude-SearchBot`, `PerplexityBot`, `Bingbot` (Copilot; also much of ChatGPT's retrieval). Training bots (`GPTBot`, `ClaudeBot`, `Google-Extended`) don't decide citability. Report each separately.
- Search Console **"Search generative AI"** setting (per site, since 2026-08-31) isn't set to exclude.

If eligibility fails, report it as Critical and stop scoring the rest of that page.

### 2. Answer-first structure (20%) [A: clear sections/headings; C: position]
- Each key question has a heading, and the **first 1-2 sentences under it answer it directly**; detail follows.
- Definitions use "X is…"; processes use ordered lists; comparisons use real `<table>`s.
- Front-load the main answer. Third-party citation studies find most AI citations come from the top part of pages [C].
- Don't split one coherent page into many thin ones (Google: no chunking).

### 3. Question coverage (15%) [A: helpfulness; C: fan-out]
- AI Mode and similar engines expand a query into sub-questions ("query fan-out") [A: Google describes the technique; C: studies correlate coverage with citations].
- For the page's main query, list the obvious follow-up questions (cost, how, vs alternatives, risks, "near me", timing) from People Also Ask, autocomplete, Search Console queries, sales/support questions, and Reddit threads.
- The page (or pages it links to) should answer the follow-ups a real reader would have. Coverage means useful completeness, **not** a page per variant.

### 4. Citable evidence (20%) [B]
The GEO study (Aggarwal et al., KDD 2024) measured what increases a source's share of generative answers:
- **Quotation addition** (credible quotes): about +41% (position-adjusted word count)
- **Statistics addition**: about +33%
- **Cite sources**: about +27%; fluency optimization about +29%
- **Keyword stuffing**: about −9%, worse than doing nothing
- Lower-ranked sources gained most (e.g. +115% for rank-5 sources with "cite sources").

So check for: specific numbers with a named source and date; quotes from named experts; links to primary sources; original data (your own measurements, calculations, surveys), which is the most defensible asset. **Never fabricate statistics or quotes to satisfy this criterion.**

### 5. Trust & entity clarity (15%) [A: E-E-A-T and people-first guidance]
- Named author with a relevant bio, or a clear organizational owner; an About/methodology page for data claims.
- `Organization`/`Person` markup with `sameAs` to real profiles (entity clarity, not a ranking hack).
- Consistent brand name, NAP, and descriptions across the site and major profiles.
- Claims match what the product and data actually support.

### 6. Freshness (10%) [A: dates/accuracy; C: citation lift]
- Visible "updated" date and `dateModified` that change only with real content changes.
- Time-sensitive facts (prices, rates, laws, years) are current. Third-party studies report recently updated pages earn more AI citations [C], so schedule refreshes for pages that answer time-bound questions.
- Never bump dates without changing content.

## Measure (don't guess)

| Surface | Tool | What it gives |
|---|---|---|
| Google AI Overviews / AI Mode / Discover AI | Search Console → **Generative AI performance** report (worldwide since 2026-08-31) | Impressions by page, country, device, date. **No clicks.** |
| Microsoft Copilot & partners | Bing Webmaster Tools → **AI Performance** (since 2026-02-11) | Citations per URL, **grounding queries**, trends. |
| ChatGPT / Perplexity / Claude | Manual prompt panel (below) or `seo-dataforseo` LLM-mention tools | Presence and cited URLs; sample-based. |
| All | Analytics referrers (chatgpt.com, perplexity.ai, copilot.microsoft.com, gemini.google.com, claude.ai) | Visits from AI answers. |

**Prompt panel:** 20-50 real questions from Search Console, PAA, and sales/support calls. Run each on each platform monthly, logged-out where possible, and record whether you're cited, which URL, and which competitor was cited instead. Answers vary run to run, so treat single results as samples, not rankings.

## Implement (with `seo-codebase` when working in a repo)

1. Fix eligibility blockers (rendering, snippet directives, robots groups).
2. Rewrite intros under key headings to answer first, keeping the voice.
3. Add sourced statistics, named quotes, and primary-source links **where true data exists**; surface original data (calculators, datasets, first-hand tests) in plain HTML text, not only in images or JS widgets.
4. Add missing follow-up answers to the page or to linked pages; add comparison tables.
5. Add visible author/updated info and entity markup.
6. Set up measurement (both reports plus the prompt panel) and a refresh schedule.

## Output

Generate `AEO-ANALYSIS.md` (in `docs/seo/` when working in a repo):
1. **AEO Score: XX/100** per page, plus the site average
2. Eligibility table per platform (bot allowed? snippet allowed? indexed?)
3. Per-page findings with grade tags [A/B/C] and the specific passage to change (quote it)
4. Question-coverage gaps (question → where to answer it)
5. Evidence gaps (claims without sources; places where original data could go)
6. Measurement setup and the prompt panel list
7. Top 5 changes ranked by impact, each with "how we'd know it worked"

## Error Handling

| Scenario | Action |
|---|---|
| Page not indexed or blocked | Report eligibility as Critical; route to `seo-indexing` / `seo-technical`. |
| No Search Console or Bing access | Recommend setup; use the prompt panel as a stopgap and say it's sample-based. |
| User asks for "AI hacks" (hidden text for LLMs, prompt injection in pages, fake stats, mass Q&A pages) | Decline; explain it violates Google spam policies and can harm trust with every engine. |
| Content has no real data to cite | Recommend creating original data (surveys, calculations, tests) rather than inventing numbers. |
