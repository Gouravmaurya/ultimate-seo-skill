# Framework Recipes

How to implement `page-rules.md` in each stack. Detect the framework and
version first (Phase 1), then use only the matching section. If the project
already has a helper for something below, extend it instead.

Every recipe covers: metadata, canonical, robots meta, JSON-LD, sitemap,
robots.txt, 404/redirects, caching for dynamic routes, and the main pitfall.

---

## Next.js, App Router (13.4+; notes for 15/16)

- **Metadata:** `export const metadata` (static) or `export async function generateMetadata({ params })` (dynamic). In 15+, `params` is a Promise: `const { id } = await params`.
- **Base URL:** set `metadataBase: new URL('https://www.example.com')` in the root layout so relative canonicals and OG images become absolute.
- **Canonical / robots:** `alternates: { canonical: '/path' }`, `robots: { index: false, follow: true }`.
- **Dedupe data fetches:** wrap the loader in React `cache()` so `generateMetadata` and the page share one query per request.
- **JSON-LD:** render in the page component:
  ```tsx
  <script type="application/ld+json"
    dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c') }} />
  ```
- **Sitemap:** `app/sitemap.ts` returning `MetadataRoute.Sitemap`. For large sets, use `generateSitemaps()` returning `[{ id: 0 }, { id: 1 }, …]` and `sitemap({ id })`. **In v16, `id` is a `Promise<string>`: `const id = Number(await props.id)`.** Output is served at `/…/sitemap/[id].xml`. A sitemap index isn't generated for you: list each file in robots.txt, or serve an index from a route handler.
- **robots.txt:** `app/robots.ts` (`MetadataRoute.Robots`) or `public/robots.txt`. Don't keep both.
- **404 / redirects:** `notFound()` from `next/navigation` (real 404 status); `permanentRedirect()` (308) or `redirects()` in `next.config`.
- **Caching dynamic routes:** `export const revalidate = 604800` plus `export function generateStaticParams() { return [] }` renders on first request, then serves from cache. Call `revalidatePath('/items/' + id)` (or `revalidateTag`) when data changes. If the project enables `cacheComponents` (v16), segment `revalidate` isn't used; use `'use cache'` with `cacheLife`/`cacheTag` instead. Check `next.config` first.
- **Pitfalls:** calling `cookies()`, `headers()`, or `auth()` in a public page or its `generateMetadata` makes the route dynamic, so every crawl becomes a fresh server render. Move per-viewer logic to client components or server actions. `images.unoptimized` is a quota trade-off, not a free win: document why.
- **OG images:** `opengraph-image.tsx` with `ImageResponse` generates per-page images; they count as function invocations on some hosts.

## Next.js, Pages Router

- **Metadata:** `next/head` in each page, or a shared `<Seo>` component. `next-seo` if already installed.
- **Data:** `getStaticProps` with `revalidate: N` (ISR) and `getStaticPaths` with `fallback: 'blocking'` for large sets; return `{ notFound: true }` for missing records (real 404).
- **Sitemap:** `next-sitemap` (postbuild) if installed; otherwise `pages/sitemap.xml.ts` with `getServerSideProps` that writes XML and sets `Cache-Control`.
- **Pitfall:** `getServerSideProps` on high-volume public pages makes every crawl a server render. Prefer ISR.

## Astro

- **Metadata:** a `<BaseHead>` component in the layout taking `title`, `description`, `canonical` (`new URL(Astro.url.pathname, Astro.site)`).
- **Sitemap:** `@astrojs/sitemap`; **requires `site`** in `astro.config`. It chunks automatically and writes `sitemap-index.xml`. Filter noindexed pages with its `filter` option.
- **Rendering:** static by default. For SSR routes (adapter), set `Cache-Control` / CDN caching and `export const prerender = true` where possible.
- **404:** `src/pages/404.astro`; in SSR, `return new Response(null, { status: 404 })` or `Astro.rewrite('/404')`.
- **Pitfall:** client islands (`client:only`) don't server-render their content, so keep indexable text out of them.

## Nuxt 3/4

- **Metadata:** `useSeoMeta({ title, description, ogTitle, ogImage })` and `useHead({ link: [{ rel: 'canonical', href }] })`.
- **SEO modules:** `@nuxtjs/seo` bundles sitemap, robots, schema-org, and OG images. If it's installed, configure it instead of hand-rolling.
- **Caching:** `routeRules: { '/items/**': { isr: 604800 } }` (or `swr`) in `nuxt.config`.
- **404:** `throw createError({ statusCode: 404, statusMessage: 'Not Found' })`.
- **Pitfall:** `ssr: false` or `<ClientOnly>` wrapping main content means crawlers get empty HTML.

## SvelteKit

- **Metadata:** `<svelte:head>` in `+page.svelte`, with data from `+page.server.ts` `load`.
- **Prerender:** `export const prerender = true` for static routes; `setHeaders({ 'cache-control': 'public, s-maxage=…' })` for SSR.
- **Sitemap / robots:** `src/routes/sitemap.xml/+server.ts` returning XML; `static/robots.txt`.
- **404:** `error(404, 'Not found')` from `@sveltejs/kit`.
- **Pitfall:** `export const ssr = false` on content routes.

## Remix / React Router v7 (framework mode)

- **Metadata:** `export const meta` returning title, description, and `{ tagName: 'link', rel: 'canonical', href }`.
- **404:** in the loader, `throw new Response('Not Found', { status: 404 })` (or `throw data(null, { status: 404 })`).
- **Caching:** `export const headers` returning `Cache-Control: public, s-maxage=…, stale-while-revalidate=…`.
- **Sitemap:** a resource route `app/routes/sitemap[.]xml.ts`.
- **Pitfall:** SPA mode (`ssr: false`) ships empty HTML for content routes.

## Gatsby

- **Metadata:** the `Head` export (Gatsby 4.19+) per page; `react-helmet` only in legacy code.
- **Sitemap:** `gatsby-plugin-sitemap`; robots via `gatsby-plugin-robots-txt`.
- **Pitfall:** DSG/SSR pages need hosting that supports them; otherwise they fall back to client rendering.

## WordPress

- **If Yoast SEO, Rank Math, or SEOPress is active, it owns titles, canonicals, sitemaps, and schema.** Configure the plugin; don't hand-code duplicates in the theme (that causes duplicate canonicals and schema).
- Theme: `add_theme_support('title-tag')`; remove hard-coded `<title>`/meta from `header.php`.
- Custom schema: the plugin's filters (for example `wpseo_schema_graph`), not a second JSON-LD block.
- Caching: page cache plugin or host cache; exclude logged-in users.
- **Pitfall:** "Discourage search engines" (Settings → Reading) left on after launch; tag/archive/attachment pages indexed as thin duplicates.

## Static site generators (Hugo, Eleventy, Jekyll)

- **Hugo:** built-in sitemap and robots (`enableRobotsTXT = true`); metadata in `layouts/partials/head.html`; `.Permalink` for canonicals.
- **Eleventy:** sitemap via a template (`sitemap.njk`) iterating `collections.all`, excluding `eleventyExcludeFromCollections` pages.
- **Jekyll:** `jekyll-seo-tag` and `jekyll-sitemap` plugins.
- **Pitfall:** `baseURL`/`url` config wrong or `localhost` in production builds makes every canonical and sitemap URL wrong.

## Single-page apps (Vite + React/Vue/Angular without SSR)

This is the highest-risk setup: crawlers that don't run JavaScript see an empty shell, status codes are always 200, and every missing page is a soft 404.
- **Best fix:** move content routes to SSR/SSG (Next.js, Nuxt, Astro, React Router framework mode, Angular SSR).
- **Stopgap:** prerender content routes at build time (e.g. `vite-plugin-ssr`/Vike, `@prerenderer`, Angular prerender) and serve real 404s from the server or CDN.
- Head tags: `react-helmet-async` / `@unhead/vue` are fine **only** with prerendering or SSR.
- If client routing must stay, inject `<meta name="robots" content="noindex">` on client-side "not found" views (Google's documented SPA soft-404 workaround).

## Any stack: hosting checks

- **Vercel:** image optimization and ISR writes are metered; crawler-heavy sites should cache renders and watch the usage dashboard.
- **Netlify:** ISR through its Next.js adapter; `_headers` or `netlify.toml` (`Netlify-CDN-Cache-Control`) for caching other frameworks.
- **Cloudflare:** cache rules for HTML, bot management for scrapers; Workers for IndexNow pings.
- **Any CDN:** make sure it doesn't cache `noindex` or 404 responses as 200 pages.
