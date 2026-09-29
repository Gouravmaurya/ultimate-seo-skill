# AEO / GEO Evidence Register

Checked 2026-09-29. Re-verify grade A items before quoting them in a client
report if this date is more than 90 days old. Grade C figures are directional:
quote them with source and date, or leave them out.

## Grade A: primary documentation

| Fact | Source |
|---|---|
| Optimizing for generative AI search "is still SEO"; llms.txt/special AI files not needed for Google; don't chunk or rewrite for AI; structured data not required for generative AI; avoid scaled pages for query variations and inauthentic mentions; use the generative AI performance report. | Google, *Optimizing for generative AI features on Google Search*, updated 2026-07-10. developers.google.com/search/docs/fundamentals/ai-optimization-guide |
| Scaled content abuse includes AI-generated pages without value, scraped feeds with minimal transformation, and combining sources without added value; thin affiliate and doorway pages. | Google, *Spam policies*, updated 2026-08-28. developers.google.com/search/docs/essentials/spam-policies |
| Search generative AI performance reports (AI Overviews, AI Mode, Discover AI features): impressions by page/country/device/date; launched 2026-06-03 (UK subset), worldwide 2026-08-31; no click data. | Google Search Central Blog, 2026-06; Search Console Help answer 16984139 |
| Search Console "Search generative AI" site control (include / exclude / inherit) controls AI Overviews and AI Mode eligibility; not a ranking signal. | Google Search Central documentation on AI features; rollout reported 2026-08-31 |
| FAQ rich results no longer shown since 2026-05-07; FAQPage remains valid Schema.org and is still parsed. | Search Engine Journal, "Google Drops FAQ Rich Results From Search" (2026) |
| `OAI-SearchBot` decides ChatGPT search inclusion; `GPTBot` is training; `ChatGPT-User` is user-triggered ("robots.txt rules may not apply"); robots changes take about 24 hours. | OpenAI, developers.openai.com/api/docs/bots |
| Anthropic user agents: `ClaudeBot` (training), `Claude-SearchBot` (search quality), `Claude-User` (user requests); all honor robots.txt. | Anthropic, support.claude.com/en/articles/8896518 |
| Google discovers links only from `<a href>`; include `noindex` in initial HTML; SPA soft-404 handling. | Google, *JavaScript SEO basics*, updated 2026-03-04 |
| Bing Webmaster Tools AI Performance: Copilot citations per URL and grounding queries; public preview 2026-02-11. | Bing Webmaster Blog, February 2026 |
| IndexNow: up to 10,000 URLs per POST; key 8-128 characters; response codes 200/202/400/403/422/429. | indexnow.org/documentation |

## Grade B: peer-reviewed

| Finding | Source |
|---|---|
| On GEO-bench, relative visibility gains (position-adjusted word count): Quotation Addition +41%, Statistics Addition +33%, Fluency +29%, Cite Sources +27%, Technical Terms +19%, Easy-to-Understand +14%, Authoritative +12%, Unique Words +6%, Keyword Stuffing −9%. Lower-ranked sources benefit most (Cite Sources: +115% for rank-5; top-ranked −30%). Effects vary by domain. | Aggarwal, Murahari, Rajpurohit, Kalyan, Narasimhan, Deshpande. *GEO: Generative Engine Optimization.* KDD 2024. arXiv:2311.09735 |

Caveat: measured on a research generative engine and benchmark (2023-24), not on the current production AI Overviews, ChatGPT, or Perplexity. The direction (evidence and sources help; stuffing hurts) matches Google's guidance; the exact percentages don't transfer.

## Grade C: industry studies (directional)

| Claim | Source (as reported) | Note |
|---|---|---|
| Brand mentions (esp. YouTube) correlate more with AI visibility than backlinks. | Ahrefs, Dec 2025, 75k brands | Correlation, not causation. Google warns against inauthentic mentions. |
| About 44% of AI citations come from the first 30% of a page. | SE Ranking | Supports answer-first structure, which is good UX anyway. |
| Content updated in the last 3 months is more likely to be cited. | SE Ranking, 1.3M citations | Refresh real content; don't bump dates. |
| AI Mode and AI Overviews cite the same URLs only about 13.7% of the time. | Ahrefs, 540k query pairs | Treat them as separate surfaces in measurement. |
| ChatGPT search results overlap heavily with Bing's index; being in Bing matters. | Multiple 2026 industry analyses | Direction consistent; exact overlap figures vary by study. |
| Citation half-life of about 4.5 weeks after IndexNow submission. | Astiva AI, 53-day Bing grounding study | Single study. |
| Share of AI Overview citations from top-10 organic pages fell during 2025-26. | Various 2026 industry reports | Figures conflict between studies; don't quote a single number. |

## Rejected or unsupported (don't recommend)

- llms.txt as a ranking/citation lever for Google (Google: ignored). For other engines it's unproven; optional at most.
- "Chunking" pages into 130-170-word blocks as a requirement (Google: not needed). Short, clear sections are fine for readers.
- Passage "cosine similarity" thresholds for citations: vendor claims with no reproducible method.
- Generating pages per query variation, fake Q&A pages, hidden text for LLMs, and prompt injection in page content: spam-policy risks.
- The Google Indexing API for non-job, non-livestream pages: against its terms.
