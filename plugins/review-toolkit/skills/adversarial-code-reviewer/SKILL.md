---
name: adversarial-code-reviewer
description: Adversarial code review that breaks the self-review blind spot by forcing four hostile perspective shifts before rendering a verdict. Use when reviewing a diff or PR before merge, when a review feels like it's about to be a rubber stamp, when Claude just said "looks good" and a second, harsher opinion is wanted, or after a long session where fatigue may be hiding bugs. Covers correctness/robustness, maintainability, security, and production operability (logging, alerting, rollback safety) — the last of which most adversarial reviewers skip.
---

# Adversarial Code Reviewer

When you review code you just wrote (or just read), your judgment shares the same mental model that produced it — you notice what you expected to see, not what's actually there. This skill breaks that by forcing four hostile personas, each with a different fear and a different definition of "broken," and requiring every one of them to find something. There is no "LGTM" exit.

## When To Use

- Before merging any PR, especially one with no human reviewer
- When a review is about to be a rubber stamp ("looks fine to me")
- After a long session — fatigue produces blind spots this compensates for
- On security- or reliability-sensitive code: auth, payments, data access, anything a pager could go off for
- Whenever something "feels off" and that instinct deserves fifteen more minutes

## Review Workflow

### 1. Gather the changes

- No target given → `git diff` (unstaged) + `git diff --cached` (staged). If both are empty, `git diff HEAD~1`.
- A ref/range given → `git diff <ref>`.
- A specific file given → read the **entire file**, not just recent changes — bugs live in how new code interacts with what's already there.
- A PR number given (and `gh` available) → `gh pr diff <number>` plus `gh pr view <number>` for description/context.

If there's nothing to review, say so and stop.

### 2. Read for context, not just diff

For every file touched: read the full file, not only the changed hunks. Identify the change's purpose (bug fix / feature / refactor / config) and note any project conventions (CLAUDE.md, linter config, surrounding style) — a finding that ignores an established convention the codebase already solved is a weak finding.

### 3. Run all four personas

Each one below must produce **at least one finding**. If a persona comes up empty, it looked too gently — go back and look harder, or (only if the code is genuinely airtight) state the single most fragile assumption it depends on.

Do not hedge findings ("this might possibly be a minor concern..."). State the failure directly: "this throws when `user` is undefined," not "this could potentially cause issues."

---

### Persona 1 — The Saboteur

**Mindset:** "I am trying to break this in production, right now, on purpose."

Look for: unvalidated input, state that can go inconsistent, concurrent access without synchronization, error paths that swallow exceptions or return a misleading result, assumptions about data shape/size/availability that a real caller will violate, off-by-ones, resource leaks (handles, connections, listeners, subscriptions).

Ask per function: "What's the worst input I could hand this?" Per external call: "What happens when it fails, hangs, or returns garbage?" Per state mutation: "What if this runs twice? Concurrently? Never?" Per conditional: "What if neither branch is actually correct?"

---

### Persona 2 — The New Hire

**Mindset:** "I joined this team an hour ago and have to modify this code in six months with zero memory of this conversation."

Look for: names that don't communicate intent, logic that requires opening three other files to follow, magic numbers/strings, a function whose name says X but that also does Y and Z, missing types that force call-chain tracing, style that drifts from its surroundings, tests that assert implementation details instead of behavior, comments that restate *what* instead of explaining *why*.

Read each changed function as if seeing the codebase for the first time. Trace one path end-to-end and count how many files it takes. If the answer relies on knowledge "the author had but the reader won't," that's the finding.

---

### Persona 3 — The Security Auditor

**Mindset:** "This will be attacked. My job is to find the hole before someone else does."

| Category | Look for |
|---|---|
| Injection | User input reaching a query, command, or template without parameterization/escaping |
| Broken auth | Hardcoded credentials, missing auth checks on a new endpoint, tokens in URLs/logs |
| Data exposure | Sensitive data in error messages/logs/responses; missing encryption in transit or at rest |
| Insecure defaults | Debug mode on, permissive CORS, wildcard permissions, default passwords |
| Access control | IDOR (can user A reach user B's data?), missing role checks, privilege escalation paths |
| Dependency risk | New dependency with a known CVE, pinned to a vulnerable version, unnecessary transitive deps |
| Secrets | Keys/tokens/passwords in code, config, comments — including "temporary" ones |

For every trust boundary crossed (user input, API call, database, filesystem, env var): is input validated, is output sanitized, is least privilege followed? Could an authenticated user escalate through this change? Does it expose new attack surface?

---

### Persona 4 — The On-Call Engineer

**Mindset:** "I will be paged for this at 3am with zero context. Can I find out what broke, or am I flying blind until someone who remembers writing this wakes up?"

Look for: error messages that don't say what actually failed ("Something went wrong"), a new failure-prone path (external call, parse, state transition) added with no log line around it, no correlation/request ID threading through the logs, a caught exception that's swallowed with no trace left behind, a new external dependency with no metric or alert wired to it, a risky behavior change with no flag/kill switch — meaning it can't be turned off without a redeploy, a schema or config change with no way back.

Ask: "If this fails in prod, what single log line tells me why, and how fast do I find it?" "Is there a metric that would page someone before a customer notices?" "Can this be turned off without a deploy?" If the answer to any of these is "read the source and hope," that's the finding.

---

### 4. Deduplicate and synthesize

Merge findings multiple personas caught independently. A finding two or more personas independently flagged is promoted one severity level — that convergence is a strong signal, not a coincidence.

## Severity

| Severity | Meaning | Action |
|---|---|---|
| **CRITICAL** | Data loss, security breach, or production outage. | Block merge. |
| **WARNING** | Will likely cause a bug in an edge case, degrade performance, or confuse the next maintainer. | Fix, or explicitly accept the risk with a stated reason. |
| **NOTE** | Style or minor improvement. | Author's discretion. |

**Promotion rule:** caught by 2+ personas → bump one level (NOTE → WARNING → CRITICAL).

## Output Format

```markdown
## Adversarial Review: [what was reviewed]

**Scope:** [files/lines/type of change]
**Verdict:** BLOCK / CONCERNS / CLEAN

### Critical Findings
[blocks the merge]

### Warnings
[should fix before merge]

### Notes
[nice to fix]

### Summary
[2-3 sentences: overall risk profile, and the single most important thing to fix]
```

**Verdict:** BLOCK on any CRITICAL. CONCERNS on 0 critical + 1+ warnings. CLEAN only when there are zero criticals and zero warnings (notes alone don't block CLEAN).

## Breaking the Self-Review Trap

You're usually reviewing code you just wrote or just read — your weights formed the same mental model that produced it, so it will look correct because it matches what you expected. To counter that:

1. Read **bottom-up** (last function first, work backward) — it breaks the narrative flow that makes code feel inevitable.
2. State each function's contract *before* reading its body. Does the body actually match what you predicted?
3. Assume every variable is null/undefined until the code proves otherwise.
4. Assume every external call fails, hangs, and returns garbage — in that order.
5. Ask: "If I deleted this change entirely, what would break?" If the honest answer is "nothing," the change itself may be the finding.

## Anti-Patterns (things this skill must never do)

| Anti-pattern | Why it's wrong |
|---|---|
| "LGTM, no issues" | If nothing was found, the review didn't look hard enough. Every change carries at least one risk, assumption, or improvement. |
| Cosmetic-only findings | Reporting whitespace while missing a null dereference is worse than no review. Substance before style. |
| Hedged findings | "This might possibly be a concern" — no. State what breaks and when. |
| Restating the diff | "This adds an auth check" is not a finding. What's wrong with the auth check? |
| Skipping test gaps | New logic without new tests is always a finding. |
| Reviewing only changed lines | Bugs live in how new code interacts with existing code — read the full file. |
| Treating this as a substitute for domain-expert or security-specialist review | This catches broad classes of issues fast; it doesn't replace a deep security or architecture review on genuinely high-stakes code. |

## Final Deliverable

One structured review in the Output Format above — verdict first, findings ranked most-severe-first, ending with the single most important fix. Nothing else; this is meant to be read in under a minute and acted on immediately.
