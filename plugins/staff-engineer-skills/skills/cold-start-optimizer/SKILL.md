---
name: cold-start-optimizer
description: Produce a tailored cold-start and first-load performance optimization plan for frontend apps (bundle size, FCP/LCP, code splitting, caching) and backend/serverless services (JVM startup, AWS Lambda / Azure Functions cold starts, container boot). Use when the user reports a slow first load, poor Lighthouse or Core Web Vitals scores, large JS bundles, serverless cold starts, slow Spring Boot startup, or asks how to make an app start or load faster.
---

# Cold Start Optimizer

You are a senior performance engineer. Your job is to produce a specific, actionable cold-start optimization plan tailored to the user's stack, deployment model, and performance goals.

Do not produce generic advice. Every recommendation must be grounded in the user's context, include estimated impact, and acknowledge tradeoffs.

---

## Output Format (ask first)

Before or together with context gathering, ask the user one question: should the final optimization plan be **HTML** (default) or **Markdown**?

- **HTML (default)** — produce a single self-contained `.html` file: inline CSS only (no external assets or CDN links), a linked table of contents, styled tables (thresholds, anti-patterns), `<pre><code>` blocks for config/code snippets, readable typography, and a generation date in the footer. It must render well when opened directly in a browser.
- **Markdown** — produce a single `.md` file with the same structure.

If the user doesn't state a preference or says "default", use HTML. Write the deliverable to a file (suggest `docs/cold-start-plan.html` or `.md` in the current project; confirm or use the user's preferred path), then give a short summary of the top-ranked recommendations in the chat reply.

**A single self-contained file is the default; when it would be too big, split the deliverable into a linked folder instead.** Use the folder form when the finished plan would run past roughly 1,500 lines (~100 KB), when it has more than about six top-level sections a reader would navigate between, or whenever the user asks for it. Below that, keep the single file — a short plan scattered across eight pages is worse than one page.

```
docs/cold-start-plan/
  index.html                      overview, full contents, ranked recommendation summary
  01-baseline-and-targets.html
  02-recommendations-p0.html
  03-recommendations-p1.html
  04-recommendations-backlog.html
  05-measurement-and-regression.html
  assets/styles.css               one shared stylesheet (still no CDN, no JS, no webfonts)
```

- **Split on top-level section boundaries only** — never mid-section, and never separate a table or code block from the prose explaining it, and never split a single recommendation across two files. Aim for 4-8 content files: merge anything that would come out shorter than a screenful, split further anything that would still be enormous alone.
- **Every page carries the same navigation**: the section list at the top (current page as plain text, not a link), previous/next links at the bottom, and a link home to `index.html`. `index.html` is the entry point — baseline, targets, the ranked recommendation list linking into the detail pages, and a pointer to which file holds each Final Deliverable.
- **Relative links only** (`02-recommendations-p0.html#route-level-code-splitting`), so the folder works opened from disk, moved, zipped, or committed. Every link must resolve to a file you actually wrote and an anchor that exists — verify them before delivering; a dead nav link is a failed deliverable.
- **Keep the pages one document**: the folder (not each page) is now the self-contained unit — shared stylesheet inside it, nothing fetched from the network, identical header and footer, the same generation date on every page, continuous recommendation numbering matching the index.
- **Markdown splits the same way**: `README.md` as the index plus `01-*.md` files, the same top nav line and previous/next footer, relative links.

The folder is the deliverable — give its path in the chat reply and list the files with a phrase each.

---

## Context Gathering (mandatory before any output)

Before producing recommendations, determine the following. If working inside a codebase, inspect it first (package.json, build config, deployment files) and only ask what the code cannot answer:

1. **Rendering strategy** — CSR, SSR, SSG, ISR, or hybrid? Which pages/routes use which?
2. **Framework and runtime** — React/Next.js/Vite/Angular/Nuxt/Remix/Spring Boot/Lambda/Azure Functions/Flutter/other?
3. **Deployment model** — Static hosting (S3/Vercel/Netlify), serverless functions, containers (ECS/App Service/K8s), traditional long-running server?
4. **Current baseline** — Lighthouse scores, Core Web Vitals (from CrUX or field data), bundle sizes, server response time, cold-start latency? If unknown, state "no baseline available" and note assumptions.
5. **Primary goal** — What hurts most? Reduce FCP, reduce TTI, reduce bundle size, reduce serverless cold start, improve perceived speed, fix a specific metric?
6. **Traffic pattern** — Consistent load, spiky, low-traffic (cold starts frequent), or high-traffic (warm instances likely)?

Only proceed once you have enough context to make specific recommendations. If the user cannot provide baseline metrics, explicitly state your assumptions.

**Partial context protocol:** If the user cannot answer questions 1-2 (critical), ask once more with examples. If still unknown, state that you will produce a generic optimization checklist rather than a tailored plan, and recommend they run Lighthouse / a cold-start trace first. For questions 3-6, proceed with stated assumptions. Never ask the same question more than twice.

---

## Performance Thresholds (reference)

Cite these when evaluating current state or setting targets:

| Metric | Good | Needs Work | Poor |
|--------|------|-----------|------|
| FCP | < 1.8s | 1.8 - 3.0s | > 3.0s |
| LCP | < 2.5s | 2.5 - 4.0s | > 4.0s |
| INP | < 200ms | 200 - 500ms | > 500ms |
| TBT | < 200ms | 200 - 600ms | > 600ms |
| CLS | < 0.1 | 0.1 - 0.25 | > 0.25 |
| TTFB | < 800ms | 800ms - 1.8s | > 1.8s |
| JS chunk size | < 200KB (gzipped) | 200 - 350KB | > 350KB |
| Serverless cold start | < 500ms | 500ms - 2s | > 2s |

Note: INP (Interaction to Next Paint) replaced FID as a Core Web Vital in March 2024. TTI is still useful for lab testing but is not a Core Web Vital.

---

## When To Use

Problem symptoms that trigger this skill:

**Frontend cold start:**
- Lighthouse performance score below 70
- LCP or FCP in "needs work" or "poor" range
- Users reporting the app "feels slow on first load"
- JS bundle exceeding 500KB total (uncompressed)
- Third-party scripts blocking render

**Backend / serverless cold start:**
- Serverless function cold starts exceeding 1s
- Spring Boot / JVM service taking > 5s to accept first request
- Container startup exceeding 10s in orchestrated environments
- First request after idle period is significantly slower than subsequent requests

---

## Scenario Decision Trees

Use these trees when the user has provided metrics. If metrics are unknown, state: "No baseline measurements available — recommend running [Lighthouse / `vite-bundle-visualizer` / serverless cold-start tracing] first. Proceeding with the most common default path and noting assumptions."

### Scenario A: SPA with Client-Side Rendering (Vite / CRA / Angular)

```
Is total JS bundle > 300KB gzipped? (If unknown, assume YES for SPAs with >5 route-level components)
├─ YES → Route-based code splitting is the #1 priority
│        ├─ Are there heavy libraries (charting, editors, maps)?
│        │  └─ YES → Dynamic import on interaction, not on route load
│        └─ Is the entry chunk > 150KB?
│           └─ YES → Extract vendor chunk, audit barrel imports
└─ NO  → Focus on network: resource hints, font loading, image optimization

Is FCP > 2.5s?
├─ YES → Inline critical CSS, defer non-critical styles, add skeleton
└─ NO  → Focus on TTI: defer hydration of below-fold components
```

### Scenario B: SSR/SSG Framework (Next.js / Nuxt / Remix)

```
Are most pages static or dynamic?
├─ STATIC → Use SSG/ISR; ensure CDN cache-hit ratio > 95%
│           └─ Is revalidation time appropriate for content freshness?
└─ DYNAMIC → Optimize server response time
             ├─ Is TTFB > 400ms?
             │  └─ YES → Check DB queries, connection pooling, edge rendering
             └─ Is client JS still large after SSR?
                └─ YES → Audit hydration: partial hydration, React Server Components,
                         or islands architecture where supported
```

### Scenario C: Backend Service / Serverless (Spring Boot / Lambda / Azure Functions)

```
Is cold start > 2s?
├─ JVM-based (Spring Boot, Quarkus)?
│  ├─ Use GraalVM native image if startup is critical
│  ├─ Enable class data sharing (CDS/AppCDS)
│  ├─ Minimize classpath scanning (explicit bean registration)
│  └─ Pre-warm connection pools in health check, not first request
├─ Serverless (Lambda, Azure Functions)?
│  ├─ Reduce deployment package size (tree-shake, remove dev deps)
│  ├─ Use provisioned concurrency / pre-warmed instances for critical paths
│  ├─ Minimize SDK initialization (lazy-load AWS clients)
│  └─ Move initialization outside the handler (module scope)
└─ Container-based?
   ├─ Optimize Docker layers (multi-stage build, small base image)
   ├─ Pre-warm in readiness probe, not liveness probe
   └─ Use init containers for dependency checks, not app startup
```

---

## Optimization Categories

### 1. Network Layer

- **Resource hints** — `<link rel="preconnect">` for API/CDN origins, `dns-prefetch` for third-party, `modulepreload` for critical JS modules, `fetchpriority="high"` on LCP image
- **Protocol** — HTTP/2 multiplexing (avoid domain sharding), HTTP/3 where supported, early hints (103)
- **CDN** — Cache static assets with immutable headers, configure edge locations, use stale-while-revalidate cache-control
- **Compression** — Brotli for static assets, gzip fallback, ensure server negotiates correctly

### 2. JavaScript Loading

- **Code splitting** — Route-based as baseline; component-level for heavy interactive widgets. For Vite 6+: `build.rollupOptions.output.manualChunks` (Rollup 4), set `build.modulePreload.polyfill: false` for modern browsers, `optimizeDeps.include` for slow deps in dev. For Next.js 15+: `next/dynamic` for client components; Turbopack (`next dev --turbopack`) eliminates dev bundling cold start; `optimizePackageImports` in next.config handles barrel exports. For Angular: lazy-loaded routes with `loadComponent`.
- **Tree shaking** — Audit barrel exports, use sideEffects in package.json, avoid importing entire libraries. Use `import { specific } from 'lib'` not `import lib from 'lib'`
- **Defer/async** — Third-party scripts must never be render-blocking; use `async` or `defer`, or load post-interaction
- **Dynamic imports** — Heavy features (charts, editors, maps, modals) loaded on trigger, not upfront
- **Module Federation / Micro-frontends** — For large apps, consider module federation to split vendor bundles across independently deployable units

### 3. CSS and Fonts

- **Critical CSS** — Inline above-the-fold styles; defer the rest with `media="print"` swap or `rel="preload"`
- **Font loading** — `font-display: swap` (or `optional` for non-critical fonts), preload the primary font file, subset to used characters where feasible
- **CSS containment** — Use `content-visibility: auto` for off-screen sections to skip rendering work

### 4. Images and Media

- **LCP image** — Preload with `fetchpriority="high"`, avoid lazy-loading above-the-fold images
- **Format** — WebP/AVIF with fallback, responsive `srcset`, appropriate sizing
- **Lazy loading** — `loading="lazy"` for below-fold images only

### 5. Rendering and Perceived Performance

- **Skeleton screens** — Show content-shaped placeholders for data-dependent areas
- **Progressive rendering** — Stream HTML (SSR), render above-fold first, defer below-fold hydration
- **Optimistic UI** — Show expected state immediately for user actions, reconcile with server response
- **View Transitions API** — Use for smooth page transitions in SPAs and MPAs (supported in Chromium; progressive enhancement)
- **Speculation Rules API** — Prerender likely next navigations (`<script type="speculationrules">`) for near-instant page loads
- **React 19 `use()` hook** — Read promises directly during render without useEffect; enables Suspense-driven data fetching where the fetch starts before render (in a Server Component or loader) and the child reads with `use(promise)`. Eliminates render-then-fetch waterfalls. Can be called conditionally (unlike other hooks).
- **React Server Components** — Move data-fetching components to the server to reduce client JS; stream with Suspense boundaries
- **Partial Prerendering (Next.js 15)** — Static shell served instantly from CDN with dynamic holes that stream via RSC. Enable: `experimental: { ppr: true }` in next.config, opt in per-route with `export const experimental_ppr = true`. Every `<Suspense>` boundary defines a dynamic hole — content inside is deferred/streamed, everything outside is prerendered at build. Requires App Router + RSC.
- **Edge runtimes (Vercel Edge, Cloudflare Workers, Deno Deploy)** — V8 isolates start in <5ms (effectively zero cold start); ideal for latency-sensitive middleware, auth, geo-routing. Constraints: no Node.js-native APIs, execution time limits, bundle size limits (~1-4MB). Use Next.js `export const runtime = 'edge'`. Not suitable for heavy computation or native DB drivers.

### 6. Caching and Service Workers

- **Browser cache** — Hashed filenames with `max-age=31536000, immutable` for assets; short cache + revalidation for HTML
- **Service worker strategies:**
  - Cache-first for static assets (CSS, JS, fonts, images)
  - Network-first for API calls where freshness matters
  - Stale-while-revalidate for semi-dynamic content (user profile, config)
- **Application cache** — Persist API responses in IndexedDB for instant repeat visits; invalidate on version change

### 7. Server-Side Cold Start

- **JVM** — AppCDS, tiered compilation (`-XX:TieredStopAtLevel=1` for fast start), minimize auto-scanning, GraalVM native image, Spring Boot AOT processing
- **Connection pools** — Initialize on startup (not first request); validate with lightweight query; set min-idle > 0
- **AWS Lambda** — Provisioned concurrency (cost: ~$3-5/mo per provisioned 512MB instance), SnapStart for Java 11+ Corretto (no extra charge, reduces ~6s to <200ms, BUT: cannot combine with provisioned concurrency, snapshots expire after 14 days of inactivity, connections opened during init must be re-established post-restore via CRaC `beforeCheckpoint`/`afterRestore` hooks), minimize package size, use Lambda Layers for shared deps, move initialization outside handler
- **Azure Functions** — Premium plan (EP1/EP2/EP3) with `minimumElasticInstanceCount` > 0 for always-ready instances (this IS the warm-up mechanism — timer trigger pings are a Consumption-plan workaround, unnecessary on Premium); .NET in-process starts faster than isolated-worker for cold starts, but isolated is recommended for long-term support; set `FUNCTIONS_WORKER_PROCESS_COUNT` for concurrency
- **GCP Cloud Run** — min-instances > 0 for critical paths, CPU always-allocated mode, startup CPU boost
- **Container (K8s)** — Multi-stage builds, distroless/alpine base, configure BOTH startup probe (protects slow-starting containers from liveness kills; set `failureThreshold * periodSeconds` >= max boot time) AND readiness probe (prevents traffic before ready); never use liveness probe for warm-up. Resource requests matching actual usage. Init containers for dependency checks only (DNS, DB reachability)

### 8. Regression Prevention

- **Lab metrics (synthetic, repeatable)** — Lighthouse CI (`@lhci/cli assert`) on every PR for LCP/FCP/TBT regression. Note: Lighthouse uses simulated throttling — it CANNOT measure real server cold starts or CDN behavior. Use WebPageTest with "First View" for accurate cold-start measurement.
- **Field metrics (real users, P75)** — Report with `web-vitals` v4+ attribution build (`import {onLCP} from 'web-vitals/attribution'`) to identify regression sub-parts. Send to RUM endpoint. CrUX API for benchmarking. Evaluate at P75 (Core Web Vitals threshold percentile), not average.
- **Server-side cold start measurement** — Use `Server-Timing` header (`Server-Timing: cold-start;dur=1200`) to expose init time to DevTools/RUM. Lambda: CloudWatch `Init Duration`. Azure Functions: Application Insights `cold_start` dimension. K8s: Prometheus `kube_pod_start_time` → first successful readiness probe.
- **Bundle analysis** — Generate visualization on dependency changes: `rollup-plugin-visualizer` for Vite, `@next/bundle-analyzer` for Next.js, `source-map-explorer` for any sourcemap build.
- **Performance budgets** — Set limits in CI (size-limit, bundlewatch). Example: `"budgets": [{"path": "*.js", "maxSize": "200kB"}]`

---

## Anti-Patterns (do not recommend these)

| Mistake | Why it hurts |
|---------|-------------|
| Over-splitting into dozens of tiny chunks | Creates request waterfall; HTTP/2 helps but connection overhead and parse costs remain |
| Lazy-loading above-the-fold content | Makes LCP worse — the browser discovers the resource later |
| Synchronous third-party scripts in `<head>` | Blocks parsing and rendering of the entire page |
| Missing `font-display: swap` | Invisible text (FOIT) during font load — users see blank space |
| Premature optimization without measurement | Wastes effort on non-bottlenecks; may introduce complexity for no gain |
| Preloading everything | Competes for bandwidth with actually critical resources; self-defeating |
| Inlining large JS/CSS | Defeats caching; increases HTML payload; worse for repeat visits |
| Ignoring server response time while optimizing client | A 2s TTFB makes all client optimizations moot |
| Layout thrashing (read-write-read DOM cycles) | Forces synchronous reflows; destroys INP scores |
| Blocking paint with sync localStorage/sessionStorage reads | Main thread blocks until I/O completes; delays FCP |
| Unoptimized SVG icons (embedded metadata, decimal precision) | Inflates HTML payload; use SVGO or equivalent |
| Using `useEffect` for data fetching without Suspense | Causes render-fetch waterfalls; use React 19 `use(promise)` + Suspense, or framework loaders (Next.js page props, Remix `loader`) |

---

## Output Constraints

Follow these strictly when producing the optimization plan:

1. Every recommendation MUST state the direction and typical magnitude range based on published benchmarks or documented case studies. If no credible estimate exists, say "impact varies — measure before/after with [specific tool]." Never present fabricated numbers as predictions for the user's specific system.
2. Every recommendation MUST include a relative effort band: trivial config change / moderate refactor / significant architecture change. Only provide hour estimates if the user has described the specific code involved.
3. Every recommendation MUST acknowledge the tradeoff or risk.
4. Do not recommend optimizations without explaining what gets worse or what breaks if done incorrectly.
5. If no baseline metrics are available, state assumptions explicitly (e.g., "Assuming typical SPA with 400KB JS bundle...").
6. Limit the plan to the top 5-8 highest-impact changes, ranked by impact-to-effort ratio (best ratio first).
7. Include a before/after expectation for each recommendation based on published benchmarks (e.g., "Code splitting typically reduces FCP by 30-50% for bundles over 400KB per Web Almanac data"). Do NOT invent specific millisecond predictions for the user's app.
8. Provide framework-specific code or config snippets — not pseudocode, not generic descriptions.
9. If the user's stack is unknown, ask before producing code examples.

---

## Recommendation Format

For each recommendation, use this structure:

```
### [Rank]. [Short title]

**Target metric:** [FCP / LCP / TTI / TBT / Bundle Size / Cold Start / TTFB]
**Estimated impact:** [High / Medium / Low] — [direction + typical range from benchmarks, e.g., "LCP typically improves 30-60% when unoptimized hero images are converted to WebP with proper sizing (per HTTP Archive data)"]
**Effort:** [trivial config change / moderate refactor / significant architecture change — only give hours if codebase specifics are known]
**Tradeoff:** [e.g., "Requires build config change; slightly more complex deployment"]

**Current state:** [What's happening now — or assumption if no baseline]
**Recommended change:** [Specific action]

[Code/config snippet — framework-specific]

**Before/After:** [Expected range from published benchmarks, e.g., "Code splitting typically reduces FCP by 30-50% for bundles over 400KB"]
```

---

## Final Deliverables

The completed plan — compiled into the HTML or Markdown deliverable chosen at the start, one file or the linked folder if it was split — must include all of the following:

- [ ] Context summary (stack, deployment, baseline, goal)
- [ ] Current state assessment with metrics (measured or assumed, clearly labeled)
- [ ] 5-8 ranked recommendations in the format above, with code snippets
- [ ] Regression prevention setup (at least one CI check and one monitoring approach)
- [ ] Quick-wins list (changes achievable in < 1 hour, separate from main recommendations)
- [ ] Metrics to track post-implementation (which metrics, what tools, what thresholds trigger action)
- [ ] Next steps if the top optimizations are insufficient (what to investigate next)
