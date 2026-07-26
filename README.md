# staff-engineer-skills

[![License: MIT](https://img.shields.io/github/license/tamasbege/staff-engineer-skills)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin%20Marketplace-5A67D8)](https://code.claude.com/docs/en/skills)

A Claude Code **plugin marketplace** of senior-specialist skills for the hard, production-critical parts of software engineering: backend reliability, adversarial review, and frontend craft. Add the marketplace once, then install whichever plugins you need.

## Contents

- [Plugins](#plugins)
  - [`the-hard-parts`](#the-hard-parts)
  - [`review-toolkit`](#review-toolkit)
  - [`frontend-hard-parts`](#frontend-hard-parts)
- [Installation](#installation)
- [Updates](#updates)
- [How they're invoked](#how-theyre-invoked)
- [Output format](#output-format)
- [License](#license)

## Plugins

### `the-hard-parts`

Eight senior-specialist [Agent Skills](https://code.claude.com/docs/en/skills) for backend correctness and reliability under failure, retry, and load. Each skill runs a staff-engineer workflow: it gathers context (inspecting your codebase first where possible), applies a battle-tested reference design, and hands back **both** a polished deliverable document (**HTML by default, or Markdown**, chosen by asking you at the start) **and** real implementation artifacts (code, DDL, IaC, tests) written into your project.

| Skill | What it produces | Use when |
|---|---|---|
| 📐 **api-response-normalizer** | Normalized API response contract: envelope spec, error/validation formats, pagination, versioning, migration plan, contract tests, response-helper code | Endpoints return inconsistent JSON shapes, error formats vary, clients need per-endpoint parsing |
| ⚡ **cold-start-optimizer** | Ranked cold-start / first-load optimization plan with framework-specific code + config and regression prevention | Slow first load, poor Lighthouse/Core Web Vitals, large bundles, serverless/JVM/container cold starts |
| 📬 **event-pipeline-architect** | Implementation-ready event pipeline design: event catalog, outbox pattern, retries/DLQs, schema evolution, scaling, observability, and IaC | Adding async processing, decoupling services, migrating sync → event-driven, designing broker topologies |
| 🔁 **idempotency-builder** | Complete idempotency system: key design, fingerprinting, locking, partial-failure recovery, payment provider coordination, monitoring, DDL + middleware | Duplicate charges/orders/webhooks, at-least-once delivery, retry-safety for critical actions |
| 🚦 **rate-limiter-designer** | Rate limiting & quota system: algorithm selection, atomic distributed enforcement (Redis Lua), limit matrix, 429 contract, outbound third-party limiting | Hammered endpoints, noisy-neighbor tenants, brute-force on login, plan quotas |
| 🛡️ **resilience-strategist** | Dependency failure-handling: timeout budgets, retries with jitter + retry budgets, circuit breakers, bulkheads, fallbacks, load shedding, chaos test plan | Downstream outages cascade, retry storms, missing timeouts, "how should we degrade?" |
| 🗃️ **caching-strategy-architect** | Caching architecture: placement (L1/L2/HTTP), pattern selection, TTL + invalidation design, stampede protection, consistency guarantees | High DB load, stale-after-write bugs, cache/DB disagreement, thundering herds |
| 🔐 **auth-flow-architect** | Auth architecture: OAuth2/OIDC flow selection, token matrix, refresh rotation + reuse detection, BFF pattern, RBAC/tenancy model, migration plan | Building login, sessions vs JWTs, IdP integration, token storage questions, multi-tenant auth |

**Slash commands**: every skill has an explicit launcher, plus a discovery command:

- `/hard-parts`: lists the skills and matches your problem to the right one
- `/normalize-api` · `/optimize-cold-start` · `/design-event-pipeline` · `/harden-idempotency` · `/design-rate-limits` · `/design-resilience` · `/design-caching` · `/design-auth`

### `review-toolkit`

Adversarial, persona-driven review skills: the antidote to the self-review blind spot.

| Skill | What it produces | Use when |
|---|---|---|
| 🥷 **blind-spot-breaker** | A structured review (`BLOCK`/`CONCERNS`/`CLEAN`) from four hostile personas (Saboteur, New Hire, Security Auditor, and an On-Call Engineer persona focused on production operability: logging, alerting, rollback safety) | Before merging a PR, when a review feels like a rubber stamp, or you want a genuinely harsh second opinion |

**Slash command:** `/adversarial-review` (supports `--diff <ref>`, `--file <path>`, or a PR number)

### `frontend-hard-parts`

Frontend craft and usability skills, currently expressive motion design and adversarial UX review, with more planned.

| Skill | What it produces | Use when |
|---|---|---|
| 💫 **expressive-motion-architect** | Bold interaction/motion design: spring-physics choreography, shape morphing, dynamic color moments, and emphasized typography, implemented across Compose, Flutter, and the web (filling the gap Material's own Expressive spec leaves on web) | A UI feels flat or "default Material," or you want spring animations, shape morphing, or a signature interaction moment |
| 🔎 **ux-reviewer** | An adversarial usability review (`DO NOT SHIP`/`SHIP WITH FIXES`/`SHIP`) from four user personas plus a Nielsen heuristics sweep | Reviewing a design/flow before dev handoff, or auditing a shipped screen for usability gaps |

**Slash commands:** `/frontend-hard-parts` (discovery) · `/expressive-motion` · `/ux-review`

*More plugins may be added to this marketplace over time.*

## Installation

### Option A: install as a plugin (recommended)

In Claude Code:

```
/plugin marketplace add tamasbege/staff-engineer-skills
/plugin install the-hard-parts
/plugin install review-toolkit
/plugin install frontend-hard-parts
```

This gives you one-command install, version tracking, and updates when the repo changes. The marketplace is just this repo, no separate infrastructure.

### Option B: copy the skills manually

The skills are plain `SKILL.md` folders, so you can also just copy them in:

```bash
git clone https://github.com/tamasbege/staff-engineer-skills.git
# personal (all projects), copy whichever plugin's skills you want:
cp -r staff-engineer-skills/plugins/the-hard-parts/skills/* ~/.claude/skills/
cp -r staff-engineer-skills/plugins/review-toolkit/skills/* ~/.claude/skills/
cp -r staff-engineer-skills/plugins/frontend-hard-parts/skills/* ~/.claude/skills/
# or per project, same paths into /path/to/project/.claude/skills/
```

## Updates

Every change here ships with a version bump, so installed plugins can pick it up cleanly:

```
/plugin marketplace update staff-engineer-skills
```

then update the plugin(s) from the `/plugin` menu (Manage plugins), or simply reinstall with `/plugin install <name>`, which always fetches the latest version. What changed in each release is in [CHANGELOG.md](CHANGELOG.md).

If you installed via Option B (manual copy), there is no update mechanism: `git pull` and re-copy the skill folders.

## How they're invoked

Claude Code discovers skills automatically from their `description` frontmatter: just describe your problem ("our endpoints all return different error shapes", "Lambda cold starts are killing us", "prevent duplicate charges on retry") or invoke one explicitly (e.g. `/idempotency-builder`).

## Output format

Most skills (all of `the-hard-parts`, plus `expressive-motion-architect`) ask one question up front: **HTML (default) or Markdown?** The design deliverable is written as a single self-contained file (suggested location: `docs/`) meant for humans: inline CSS, table of contents, styled tables and code blocks. Implementation artifacts (DDL, middleware, IaC, config) are additionally written to real source files when you want them applied.

`blind-spot-breaker` and `ux-reviewer` are review workflows, not design-and-build skills. They instead produce a structured verdict directly in chat (`BLOCK`/`CONCERNS`/`CLEAN` and `DO NOT SHIP`/`SHIP WITH FIXES`/`SHIP` respectively), built for fast, iterative review rather than a saved document.

## License

MIT. See [LICENSE](LICENSE).
