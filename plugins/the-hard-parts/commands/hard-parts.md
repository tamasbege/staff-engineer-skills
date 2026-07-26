---
description: List the-hard-parts skills and pick the right one for your problem
---

The user wants to know what the-hard-parts plugin offers, or isn't sure which skill fits their problem.

Show them this catalog, then ask which problem they're facing (or match their description below to a skill and offer to launch it):

| Skill | Solves | Typical symptoms |
|---|---|---|
| **api-response-normalizer** (`/normalize-api`) | Inconsistent API response shapes | Every endpoint returns different JSON; error formats vary; `response?.data?.data` chains in clients |
| **cold-start-optimizer** (`/optimize-cold-start`) | Slow first load / cold starts | Poor Lighthouse scores; big bundles; Lambda/JVM/container cold starts; "feels slow on first load" |
| **event-pipeline-architect** (`/design-event-pipeline`) | Async / event-driven design | Adding queues or events between services; retries/DLQs; sync→async migration |
| **idempotency-builder** (`/harden-idempotency`) | Duplicate execution on retry | Double charges/orders; webhook replays; at-least-once delivery; double-click submits |
| **rate-limiter-designer** (`/design-rate-limits`) | Throttling, quotas, abuse | Hammered endpoints; noisy-neighbor tenants; brute-force on login; plan quotas; third-party API limits |
| **resilience-strategist** (`/design-resilience`) | Surviving dependency failures | Downstream outage cascades; retry storms; missing timeouts/circuit breakers; "how should we degrade?" |
| **caching-strategy-architect** (`/design-caching`) | Caching that stays correct | High DB load; stale data after writes; cache stampedes; "where/how should we cache?" |
| **auth-flow-architect** (`/design-auth`) | Authentication & authorization | Building login; sessions vs JWTs; IdP integration; token storage/refresh questions; multi-tenant auth |

If the user already described their problem in the arguments below, recommend the matching skill (or combination — e.g., retry-safety usually needs idempotency-builder before resilience-strategist) and offer to invoke it now.

User's problem description (may be empty): $ARGUMENTS
