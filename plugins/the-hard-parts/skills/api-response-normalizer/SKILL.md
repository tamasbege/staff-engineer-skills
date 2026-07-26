---
name: api-response-normalizer
description: Design a normalized API response contract so every endpoint returns a predictable shape for success, errors, validation failures, and collections. Use when endpoints return inconsistent JSON shapes, error formats vary by endpoint, pagination metadata is scattered, validation errors lack field references, or the user asks to standardize API responses, design a response envelope, or define an error contract. Not for GraphQL schema design, event/message contracts, file streaming, or WebSocket frames.
---

# API Response Normalizer

You are a senior API architect. Your job is to design a normalized response contract for an API so that every endpoint returns a predictable, parseable shape for success, errors, validation failures, and collections.

## When To Use

Trigger this skill when you observe these symptoms:

- Endpoints return different JSON shapes for the same outcome (e.g., some wrap in `data`, some don't)
- Error responses vary by endpoint (string messages vs objects vs plain status codes)
- Pagination metadata lives in different places (headers, body root, nested object)
- Validation errors are unstructured or missing field references
- Clients need per-endpoint parsing logic instead of a single response handler
- Frontend code is littered with `response?.data?.data` or null-guard chains

Do NOT use this skill for: GraphQL schema design, event/message contracts, file streaming endpoints, or WebSocket frame formats.

---

## Phase 0: Output Format (ask first)

Before or together with context gathering, ask the user one question: should the final deliverable document be **HTML** (default) or **Markdown**?

- **HTML (default)** — produce a single self-contained `.html` file: inline CSS only (no external assets or CDN links), a linked table of contents, styled tables, `<pre><code>` blocks for JSON/code, readable typography, and a generation date in the footer. It must render well when opened directly in a browser.
- **Markdown** — produce a single `.md` file with the same structure.

If the user doesn't state a preference or says "default", use HTML. Write the deliverable to a file (suggest `docs/api-response-contract.html` or `.md` in the current project; confirm or use the user's preferred path), then give a short summary of the key decisions in the chat reply. Implementation code (response helpers, error classes, handlers) additionally goes into real source files where the user wants it — the document embeds copies for reading.

---

## Phase 1: Context Gathering (Mandatory)

Before producing any output, ask the user ALL of the following. Do not skip questions or assume defaults:

1. **Tech stack** — What language/framework serves the API? What consumes it? (e.g., Spring Boot + React, Express + mobile clients)
2. **Scope** — Are we normalizing the entire API, a single service, or specific endpoints? List them.
3. **Existing patterns** — Paste one current success response and one current error response so I can see the starting shape.
4. **Greenfield or migration** — Is this a new API or are there existing clients that will break if the contract changes?
5. **Pagination needs** — Does the API serve paginated lists? If yes, what volume (tens, thousands, millions of records)?
6. **Versioning** — Is there an existing versioning scheme (URL path, header, query param)? Any constraints?
7. **Special requirements** — Rate limiting, multi-tenancy, partial responses, bulk operations, long-running async tasks?

Wait for answers before proceeding. Adapt all output to the user's actual stack, scope, and constraints. If working inside a codebase, inspect it first (controllers, error handlers, existing responses) and only ask the questions the code cannot answer.

**Partial context protocol:** If the user cannot answer a question — for questions 1-3 (critical), ask once more with a concrete example. If still unknown, state that you will produce a technology-agnostic structural template and note assumptions. For questions 4-7 (nice-to-have), proceed with stated defaults. Never ask the same question more than twice.

---

## Phase 2: Reference Contract (Canonical Shapes)

Use these as structural anchors. Adapt field names and casing to the user's stack conventions.

### Success — Single Resource (Flat style — default for most REST APIs)

```json
{
  "data": {
    "id": "res_8xK2mP",
    "status": "confirmed",
    "total": 149.99,
    "currency": "EUR",
    "createdAt": "2026-03-15T10:22:00Z"
  },
  "meta": {
    "requestId": "req_abc123",
    "timestamp": "2026-03-15T10:22:01Z"
  }
}
```

### Success — Collection (Paginated)

```json
{
  "data": [
    { "id": "res_8xK2mP", "status": "confirmed", "total": 149.99, "currency": "EUR" },
    { "id": "res_9yL3nQ", "status": "shipped", "total": 89.00, "currency": "EUR" }
  ],
  "meta": {
    "requestId": "req_def456",
    "timestamp": "2026-03-15T10:23:00Z"
  },
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalItems": 843,
    "totalPages": 43,
    "hasNext": true,
    "hasPrevious": true
  }
}
```

### Error — Operational Failure

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested order does not exist.",
    "target": "orderId",
    "details": []
  },
  "meta": {
    "requestId": "req_ghi789",
    "timestamp": "2026-03-15T10:24:00Z"
  }
}
```

### Error — Validation Failure (422)

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "One or more fields failed validation.",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Must be a valid email address.",
        "rejectedValue": "not-an-email"
      },
      {
        "field": "items[0].quantity",
        "code": "OUT_OF_RANGE",
        "message": "Must be between 1 and 9999.",
        "rejectedValue": 0
      }
    ]
  },
  "meta": {
    "requestId": "req_jkl012",
    "timestamp": "2026-03-15T10:25:00Z"
  }
}
```

**Choosing a structure:**
- Use the flat style (above) unless the API serves polymorphic collections where clients need type discrimination.
- If the API uses JSON:API or requires a `type`/`attributes` split, wrap resource fields in `"attributes": {}` and add a `"type"` field. Only do this when the user's API already follows this convention or explicitly requests it.

### Framework-native error format (Spring Boot ProblemDetail / RFC 7807)

When the user's framework has a standard error format, produce THIS shape instead of the canonical `error` envelope:

```json
{
  "type": "https://api.example.com/problems/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "One or more fields failed validation.",
  "instance": "/api/v1/orders",
  "requestId": "req_jkl012",
  "timestamp": "2026-03-15T10:25:00Z",
  "errors": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Must be a valid email address.",
      "rejectedValue": "not-an-email"
    }
  ]
}
```

Equivalent applies to: ASP.NET ProblemDetails, Django REST Framework exception responses, FastAPI HTTPException with `detail` dict. Use the framework format as the base and enrich with `requestId`, `timestamp`, and structured `errors[]` where missing.

---

**Pagination strategy compatibility:**
- Offset pagination: `page`, `pageSize`, `totalItems`, `totalPages`, `hasNext`, `hasPrevious` are all valid.
- Cursor/keyset pagination: replace `page`/`totalItems`/`totalPages` with `cursor`/`nextCursor`/`previousCursor`. The `totalItems` field becomes optional (expensive COUNT query) — omit it or mark as approximate. Never require `totalItems` for cursor-based APIs.

These examples are starting points. Adapt to the user's actual needs and existing conventions.

---

## Phase 3: Build Order

Follow this sequence. Each step produces a concrete deliverable.

1. **Audit current shapes** — Using the responses the user provided in Phase 1, identify where they diverge from each other and from the canonical shape. If the user did not provide examples, skip this step and note: "No current-state audit performed — recommendations are based on the canonical shape. Provide example responses if you need migration guidance."
2. **Define the envelope** — Lock down top-level keys (`data`, `error`, `meta`, `pagination`). Justify each one. Specify which are present in which scenarios.
3. **Define success variants** — Single resource, collection, created (201 + `Location` header pointing to new resource URI), accepted async (202), no-content (204 — no body permitted per RFC 9110, document when to use). Provide a JSON example for each variant that carries a body.
4. **Define error variants** — Map each HTTP status (400, 401, 403, 404, 409, 422, 429, 500, 503) to an error code and example body.
5. **Define validation format** — Field path syntax (dot notation, bracket notation for arrays), per-field error shape, cross-field errors.
6. **Define pagination** — Choose ONE strategy (offset, cursor, keyset). Document the pagination object fields and edge cases (empty page, last page, unknown total).
7. **Define metadata** — What goes in `meta`. At minimum: `requestId`, `timestamp`. Optionally: `deprecation`, `warnings`, `processingTimeMs`.
8. **Versioning and content negotiation** — How is the contract versioned? How do clients request a version? What happens on version mismatch?
9. **Rate-limit headers** — Define standard headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`. Define the 429 body.
10. **Write shared implementation** — Response builder/helper, error classes/types, validation formatter, global error handler. In the user's actual language.
11. **Migration plan** — If existing clients exist: order endpoints by risk, define a compatibility window, specify deprecation headers.
12. **Contract tests** — Provide test cases that assert response shape (not just status code). At minimum: success, error, validation, empty list, paginated list.

---

## Input Specification

When the user provides an endpoint to normalize, collect or infer:

- HTTP method and path
- Request body schema (if applicable)
- Query parameters (filtering, sorting, pagination)
- Required headers (auth, content-type, accept, API version)
- Possible outcomes (success, not found, validation failure, conflict, etc.)

Produce the normalized response for EACH possible outcome of that endpoint.

---

## Versioning Strategy

Address these decisions explicitly:

- **Mechanism**: URL path (`/v2/orders`), header (`Accept: application/vnd.api+json;version=2`), or query param (`?version=2`). Recommend one and justify.
- **Breaking vs non-breaking**: Define what constitutes a breaking change (removing a field, changing a type, renaming a key) vs non-breaking (adding an optional field, adding a new error code).
- **Sunset policy**: How long do old versions live? How are clients notified? (Recommend `Sunset` and `Deprecation` headers per RFC 8594.)

---

## Additional Concerns (address when relevant)

### Conditional Requests & Caching
- **ETag** — Include `ETag` header on GET responses for cacheable resources. Support `If-None-Match` (304 Not Modified).
- **Last-Modified** — For time-based resources, support `If-Modified-Since`.
- **Cache-Control** — Define caching semantics per endpoint type (static data vs volatile).

### CORS
- If the API is consumed by browsers from different origins, define CORS response headers (`Access-Control-Allow-Origin`, `Allow-Methods`, `Allow-Headers`). Note whether the contract should document these or if they're handled by infra (API gateway, reverse proxy).

### Async Operations (202 Accepted)
For long-running operations, define the response shape:
```json
{
  "data": {
    "operationId": "op_xyz789",
    "status": "processing",
    "statusUrl": "/api/v1/operations/op_xyz789",
    "estimatedCompletionSeconds": 30
  },
  "meta": { "requestId": "req_mno345", "timestamp": "..." }
}
```

### Bulk Operations
When endpoints accept multiple items in a single request, define partial-success behavior:
- All-or-nothing (return 400 if any item fails)?
- Partial success (return 207 Multi-Status or 200 with per-item results)?
- Document the chosen approach in the contract.

### Health Check Response
Align with your framework's built-in health format:
- **Spring Boot**: Use Actuator `/actuator/health` (no custom shape needed)
- **ASP.NET**: Use `Microsoft.Extensions.Diagnostics.HealthChecks`
- **Express/Fastify**: Use Terminus or Lightship pattern
- **Kubernetes**: Separate `/healthz` (liveness — 200 if process alive) from `/readyz` (readiness — checks dependencies)

If no framework health system exists:
```json
{ "status": "healthy", "checks": [{ "name": "database", "status": "healthy", "responseTimeMs": 12 }] }
```
Return HTTP 200 for healthy, 503 for unhealthy. Keep it simple — orchestrators primarily use the status code.

### Infrastructure Error Passthrough
Not all error responses originate from your application. API gateways, load balancers, WAFs, and reverse proxies return their own error pages (often HTML). Address this:
- Document which HTTP statuses may arrive as non-JSON (typically 502, 503, 504)
- Require clients to check `Content-Type` header before parsing body as JSON
- If possible, configure infra (API gateway, nginx) to return JSON error bodies matching your envelope
- Add to contract test suite: "client handles non-JSON error response gracefully"

---

## Anti-Patterns (Avoid These)

| Anti-pattern | Why it's harmful | Correct approach |
|---|---|---|
| Returning 200 with `{ "success": false }` | Clients must parse the body to detect failure; HTTP semantics are ignored | Use appropriate 4xx/5xx status codes |
| Exposing stack traces in production | Security risk; leaks internals to attackers | Log internally, return only `error.code` + safe message |
| Mixing data and metadata at the same level | Clients can't generically extract the resource vs ancillary info | Separate into `data` and `meta` |
| Different error shapes per endpoint | Every consumer needs per-route error handling | Single error envelope everywhere |
| Using 404 for empty collections | An empty list is a valid result, not an absence | Return 200 with `"data": []` |
| Inconsistent null handling (some fields omitted, some explicit null, no documented rule) | Clients cannot distinguish "absent" from "not applicable" from "null" | Choose ONE strategy and document it: (a) omit absent optional fields (Jackson `@JsonInclude(NON_NULL)`) — best for dynamic clients, or (b) include null explicitly — best for strongly-typed clients. Either is valid; inconsistency is the anti-pattern |
| Pagination without stability guarantees | Concurrent writes cause skipped/duplicate items | Document consistency model; prefer cursor/keyset for large sets |
| Inventing HTTP status codes | Proxies and clients don't understand non-standard codes | Stick to registered IANA codes; use `error.code` for granularity |
| Returning `rejectedValue` with sensitive data | Leaks passwords, tokens, or PII in validation errors | Only include `rejectedValue` for non-sensitive fields; omit or mask for credentials/PII |
| Missing `Content-Type: application/json` header | Some clients default to text/html parsing; causes subtle bugs | Always set content-type explicitly on all JSON responses |
| No idempotency guidance for POST/PUT | Clients can't safely retry — duplicates possible | Reference the Idempotency-Key header pattern for mutating endpoints |

---

## Output Constraints

Follow these rules when producing deliverables:

1. Every contract definition MUST include a concrete JSON example. No abstract descriptions without a corresponding code block.
2. Scale output to the user's request scope. If they asked about 3 endpoints, do not produce a 40-page document covering every theoretical case.
3. When rules conflict, prefer client parsability over server convenience.
4. Use the user's actual field naming convention (camelCase, snake_case, kebab-case). Ask if unclear.
5. Provide a base set of error codes that map 1:1 to HTTP semantics (UNAUTHORIZED, FORBIDDEN, RESOURCE_NOT_FOUND, CONFLICT, RATE_LIMITED, VALIDATION_FAILED, INTERNAL_ERROR, SERVICE_UNAVAILABLE). These are the MINIMUM set. Encourage the user to extend with domain-specific codes as sub-codes (e.g., ORDERS.ALREADY_SHIPPED, USERS.DUPLICATE_EMAIL) — either via a `subCode` field or namespaced codes. Do not limit users to only HTTP-semantic codes.
6. If the user's framework has a standard error format (e.g., Spring's ProblemDetail / RFC 7807, Rails error hash), USE the framework format as the base and extend it with fields from our canonical shape that are missing (e.g., add a `details[]` array for validation errors within ProblemDetail's extension fields). Do NOT replace the framework format with the canonical shape — show how to enrich it.
7. Produce implementation code in the user's language. Do not give pseudocode unless the language is unspecified after asking.
8. Every migration step must be backward-compatible unless the user explicitly accepts a breaking change.

---

## Final Deliverables

Hand back exactly these artifacts (scope-adjusted to what the user actually needs), compiled into the single HTML or Markdown document chosen in Phase 0:

- [ ] **Response envelope spec** — The normalized top-level shape for success, error, and collection responses, with JSON examples
- [ ] **Status code map** — Table mapping each outcome to HTTP status + error code
- [ ] **Validation error format** — Field-level error shape with path syntax and examples for nested/array fields
- [ ] **Pagination contract** — Chosen strategy with request params and response fields
- [ ] **Implementation code** — Response helpers, error classes, global handler in the user's language
- [ ] **Migration plan** — Ordered list of endpoints to migrate (if not greenfield), with compatibility notes
- [ ] **Contract test cases** — Assertions for each response variant (success, each error type, empty collection, paginated)
- [ ] **Header contract** — Rate-limit, versioning, content-type, caching (ETag/Cache-Control), and security headers (CORS if browser-consumed)
