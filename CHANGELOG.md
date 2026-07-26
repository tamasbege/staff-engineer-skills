# Changelog

## 1.0.0 — 2026-07-26

Initial release: the `the-hard-parts` plugin, published via the self-hosted `staff-engineer-skills` marketplace.

### Skills

1. **api-response-normalizer** — normalized API response contracts (success, error, validation, pagination)
2. **cold-start-optimizer** — frontend and backend cold-start / first-load performance plans
3. **event-pipeline-architect** — production-grade event-driven pipeline design
4. **idempotency-builder** — retry-safe systems for payments, orders, and webhooks
5. **rate-limiter-designer** — throttling, quotas, and abuse protection
6. **resilience-strategist** — timeouts, retries, circuit breakers, bulkheads, fallbacks
7. **caching-strategy-architect** — cache placement, invalidation, and stampede protection
8. **auth-flow-architect** — OAuth2/OIDC flows, token lifecycle, sessions, RBAC

Every skill asks HTML (default) or Markdown for its deliverable up front, inspects the codebase before asking context questions, and hands back both a design document and real implementation artifacts (code, DDL, IaC, tests).

### Commands

- `/hard-parts` — discovery command: catalogs the skills and matches a described problem to the right one
- Per-skill launchers: `/normalize-api`, `/optimize-cold-start`, `/design-event-pipeline`, `/harden-idempotency`, `/design-rate-limits`, `/design-resilience`, `/design-caching`, `/design-auth`

### Packaging

- Multi-plugin monorepo layout: the marketplace (`staff-engineer-skills`) hosts plugins under `plugins/<name>/`, so more can be added later without restructuring.
- `the-hard-parts` is the first plugin, bundling all 8 skills and 9 commands.

### Authoring & review process

Every skill was drafted, then hardened through adversarial review (Saboteur / New Hire / Security Auditor personas) and domain-expert review (senior-backend, senior-architect, senior-security, senior-devops, cloud specialists) before inclusion. Selected fixes this process caught:

- **idempotency-builder** — pseudocode switched to `DB_NOW()` instead of the app clock to avoid cross-node clock skew; added constant-time fingerprint comparison, a `lock_version` CAS column for ABA-safe orphan-lock reclaim, and a Stripe/Adyen/PayPal provider-coordination table.
- **event-pipeline-architect** — corrected producer pseudocode that showed a contradictory dual-write instead of the outbox pattern; added a scope gate (single event vs. full pipeline vs. redesign) and IaC (Bicep/Terraform) per broker.
- **cold-start-optimizer** — fixed AWS Lambda SnapStart limitations (incompatible with provisioned concurrency, needs CRaC hooks) and Azure Functions guidance (Premium `minimumElasticInstanceCount`, not timer-trigger pings); replaced `useEffect` data-fetching anti-pattern with React 19 `use()` + Suspense.
- **api-response-normalizer** — fixed an RFC violation in the 204 No Content guidance; resolved a conflict between the canonical envelope and framework-native error formats (e.g. Spring's ProblemDetail) by enriching rather than replacing them.
- **rate-limiter-designer** — added the trusted-proxy-hop rule (a client-supplied `X-Forwarded-For` bypasses every per-IP limit otherwise); security limits (login/reset) now suppress remaining-count headers so attackers can't read a brute-force countdown; limiter state must run on `noeviction` Redis, since LRU eviction silently resets limits under memory pressure.
- **resilience-strategist** — fixed an internal contradiction between the bulkhead example (implied queueing) and the fail-fast-only policy stated elsewhere; corrected an incorrect claim that retries never reach an open circuit breaker.
- **caching-strategy-architect** — fixed a critical bug in the reference pseudocode that returned the negative-cache tombstone sentinel as if it were real data on a cache hit — the exact class of bug the skill itself warns against.
- **auth-flow-architect** — added session-fixation protection (new session ID at login, rotated on privilege change), CORS discipline for the cookie-based BFF pattern (wildcard/reflected origin plus credentials reopens CSRF), and JWKS caching with a bounded refetch-on-unknown-`kid` rule.

Full per-skill, timestamped authoring notes (including earlier draft iterations) are kept in each skill's git history.
