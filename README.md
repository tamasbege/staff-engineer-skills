# staff-engineer-skills

[![License: MIT](https://img.shields.io/github/license/tamasbege/staff-engineer-skills)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin%20Marketplace-5A67D8)](https://code.claude.com/docs/en/skills)

A Claude Code **plugin marketplace** of senior-specialist skills for the hard, production-critical parts of software engineering: backend reliability, adversarial review, and frontend craft. Add the marketplace once, then install the plugin.

## Contents

- [Skills](#skills)
  - [Backend reliability](#backend-reliability)
  - [Review](#review)
  - [Frontend craft](#frontend-craft)
- [Installation](#installation)
- [Updates](#updates)
- [How they're invoked](#how-theyre-invoked)
- [Output format](#output-format)
- [License](#license)

## Skills

Eleven senior-specialist [Agent Skills](https://code.claude.com/docs/en/skills), bundled in one plugin (`staff-engineer-skills`). Each skill runs a staff-engineer workflow: it gathers context (inspecting your codebase first where possible) and applies a battle-tested reference design. Design-and-build skills hand back **both** a polished deliverable document (**HTML by default, or Markdown**, chosen by asking you at the start) **and** real implementation artifacts (code, DDL, IaC, tests) written into your project; review skills instead produce a structured verdict directly in chat (see [Output format](#output-format)).

`/skills`: lists all eleven and matches your problem to the right one.

### Backend reliability

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

Slash commands: `/normalize-api` · `/optimize-cold-start` · `/design-event-pipeline` · `/harden-idempotency` · `/design-rate-limits` · `/design-resilience` · `/design-caching` · `/design-auth`

### Review

Adversarial, persona-driven review: the antidote to the self-review blind spot.

| Skill | What it produces | Use when |
|---|---|---|
| 🥷 **blind-spot-breaker** | A structured review (`BLOCK`/`CONCERNS`/`CLEAN`) from four hostile personas (Saboteur, New Hire, Security Auditor, and an On-Call Engineer persona focused on production operability: logging, alerting, rollback safety) | Before merging a PR, when a review feels like a rubber stamp, or you want a genuinely harsh second opinion |

Slash command: `/adversarial-review` (supports `--diff <ref>`, `--file <path>`, or a PR number)

### Frontend craft

Currently expressive motion design and adversarial UX review, with more planned.

| Skill | What it produces | Use when |
|---|---|---|
| 💫 **expressive-motion-architect** | Bold interaction/motion design: spring-physics choreography, shape morphing, dynamic color moments, and emphasized typography, implemented across Compose, Flutter, and the web (filling the gap Material's own Expressive spec leaves on web) | A UI feels flat or "default Material," or you want spring animations, shape morphing, or a signature interaction moment |
| 🔎 **ux-reviewer** | An adversarial usability review (`DO NOT SHIP`/`SHIP WITH FIXES`/`SHIP`) from four user personas plus a Nielsen heuristics sweep | Reviewing a design/flow before dev handoff, or auditing a shipped screen for usability gaps |

Slash commands: `/expressive-motion` · `/ux-review`

*More skills, and potentially more plugins, may be added to this marketplace over time.*

## Installation

### Option A: install as a plugin (recommended)

In Claude Code:

```
/plugin marketplace add tamasbege/staff-engineer-skills
/plugin install staff-engineer-skills
```

This gives you one-command install, version tracking, and updates when the repo changes. The marketplace is just this repo, no separate infrastructure.

By default this installs just for you, across all your projects (`user` scope). Two other scopes are available via `--scope`:

- `--scope project`: shared with every collaborator on a repo (writes to that repo's `.claude/settings.json`, so it's committed)
- `--scope local`: just for you, just in the current repo (writes to `.claude/settings.local.json`, gitignored)

Example: `claude plugin install staff-engineer-skills@staff-engineer-skills --scope project`

### Option B: copy the skills manually

The skills are plain `SKILL.md` folders, so you can also just copy them in:

```bash
git clone https://github.com/tamasbege/staff-engineer-skills.git
# personal (all projects):
cp -r staff-engineer-skills/plugins/staff-engineer-skills/skills/* ~/.claude/skills/
# or per project, same path into /path/to/project/.claude/skills/
```

## Updates

Every change here ships with a version bump, so an installed plugin can pick it up cleanly:

```
/plugin marketplace update staff-engineer-skills
```

then update the plugin from the `/plugin` menu (Manage plugins), or simply reinstall with `/plugin install staff-engineer-skills`, which always fetches the latest version. What changed in each release is in [CHANGELOG.md](CHANGELOG.md).

If you installed via Option B (manual copy), there is no update mechanism: `git pull` and re-copy the skill folders.

## How they're invoked

Claude Code discovers skills automatically from their `description` frontmatter: just describe your problem ("our endpoints all return different error shapes", "Lambda cold starts are killing us", "prevent duplicate charges on retry") or invoke one explicitly (e.g. `/idempotency-builder`).

## Output format

Design-and-build skills (everything under [Backend reliability](#backend-reliability), plus `expressive-motion-architect`) ask one question up front: **HTML (default) or Markdown?** The design deliverable is written as a single self-contained file (suggested location: `docs/`) meant for humans: inline CSS, table of contents, styled tables and code blocks. Implementation artifacts (DDL, middleware, IaC, config) are additionally written to real source files when you want them applied.

`blind-spot-breaker` and `ux-reviewer` are review workflows, not design-and-build skills. They instead produce a structured verdict directly in chat (`BLOCK`/`CONCERNS`/`CLEAN` and `DO NOT SHIP`/`SHIP WITH FIXES`/`SHIP` respectively), built for fast, iterative review rather than a saved document.

## License

MIT. See [LICENSE](LICENSE).
