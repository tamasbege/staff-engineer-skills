# Changelog

## 1.1.1 — 2026-07-26

- Renamed `review-toolkit`'s **adversarial-code-reviewer** to **blind-spot-breaker** — a name describing what the skill accomplishes rather than the mechanism, and distinct from its inspiration's name (see below). The `/adversarial-review` command is unchanged.
- Added explicit credit in the skill file: the three-persona structure (Saboteur, New Hire, Security Auditor) and severity-promotion mechanic are adapted from the `adversarial-reviewer` skill in Alireza Rezvani's [claude-skills](https://github.com/alirezarezvani/claude-skills) `engineering-skills` plugin (MIT licensed). This repo's version is an independent rewrite with a fourth persona (On-Call Engineer) added, not a copy of the original text.

## 1.1.0 — 2026-07-26

Two new plugins, validating the multi-plugin marketplace layout for the first time.

### `review-toolkit` (new plugin)

- **adversarial-code-reviewer** — four-persona adversarial review (Saboteur, New Hire, Security Auditor, and a new **On-Call Engineer** persona focused on production operability: logging, alerting, rollback safety — the class of finding most adversarial reviewers skip). `/adversarial-review` command.

### `frontend-hard-parts` (new plugin)

- **expressive-motion-architect** — bold interaction/motion design (spring physics, shape morphing, dynamic color, emphasized typography) across Compose, Flutter, and web. Explicitly fills the gap the installed `material-3` skill flags but doesn't solve: M3 Expressive has no implementation in `@material/web`. Complements rather than duplicates `material-3` (that skill owns tokens/compliance; this one owns the motion/interaction layer on top).
- **ux-reviewer** — adversarial, persona-driven usability review (First-Time User, Impatient Power User, Accessibility User, Anxious/Error-Prone User) plus a Nielsen heuristics sweep. Distinct from the installed `ux-researcher-designer` skill: that one generates personas/research from data; this one critiques an existing design against fixed archetypes.
- `/frontend-hard-parts` discovery command, `/expressive-motion`, `/ux-review`.

### Authoring & review

Both plugins went through the same process as 1.0.0: drafted, then adversarial review. Fixes this pass caught:

- **adversarial-code-reviewer** — the verdict rule left a gap where a single WARNING (defined as "likely to cause bugs") would still render CLEAN; tightened so CLEAN requires zero warnings, not just fewer than two.
- **expressive-motion-architect** — corrected a real technical inaccuracy: Flutter's `Path.combine` performs boolean path operations (union/difference/intersect), not shape interpolation — the prose claimed it could be used for morphing, contradicting the skill's own (correct) implementation table. Replaced with `ShapeBorderTween` / `Path.computeMetrics()`-based interpolation. Also added a browser-support caveat to the CSS `linear()` spring-easing approximation, matching the rigor already given to the shape-morph section.
- **ux-reviewer** — fixed a garbled touch-target citation that merged two different standards' numbers into one invalid "44×48px" figure; replaced with the three actual sources (WCAG 2.5.8, Apple HIG, Material) and their real values. Same verdict-threshold gap as the code reviewer, fixed the same way.

### Reference

- Added `CONTRIBUTING.md`, which states the actual security model for this repo: skill files are executable agent instructions, so every PR is read in full before merge — there is no automated scanner that reliably catches malicious prompt content.

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
