# Contributing

Thanks for considering a contribution. This repo is a Claude Code plugin marketplace, and most contributions will be either a new skill, a fix/improvement to an existing one, or a new plugin.

## Before you open a PR

**A skill file is not documentation. It's executable instructions for an AI agent with tool access.** Anyone who installs a plugin from this marketplace is trusting that its skills won't be told to do something harmful once installed. Because of that:

- Every PR is read in full, line by line, before merge, not just diffed for style.
- PRs that add unexplained obfuscated instructions, instructions to exfiltrate data, disable safety behavior, or act outside the skill's stated purpose will be rejected outright, no discussion.
- If a change is large or the intent isn't obvious from the diff, explain the "why" in the PR description. Reviewers should not have to guess.

This is the actual security model for this repo: there is no automated scanner that reliably catches malicious prompt content, so manual review is the control.

## Adding a new skill

Put it at `plugins/staff-engineer-skills/skills/<skill-name>/SKILL.md`. Match the house style used by the existing skills:

1. **YAML frontmatter** with `name` and a `description` written in third person, packed with the concrete symptoms/phrasings a user would actually say. This is what drives automatic invocation.
2. **Phase 0, output format**, for design-and-build skills that produce a shareable deliverable (the norm, e.g. everything in the backend-reliability category): ask HTML (default) or Markdown up front, before anything else. Skip this for fast, iterative review-workflow skills (like `blind-spot-breaker`/`ux-reviewer`) that instead output a structured verdict directly in chat. State which pattern your skill uses and why in the PR description.
3. **Mandatory context-gathering phase**, with an instruction to inspect the codebase first and only ask what the code can't answer, plus a partial-context protocol (ask once more, then fall back gracefully, never loop).
4. **A concrete reference example** (real schema/code/config, not abstract description) showing the depth expected of every output.
5. **An anti-patterns table** and a **testable constraints** list the output must satisfy.
6. **A final deliverables checklist.**

Read a couple of the existing `SKILL.md` files before starting, picking ones that match the kind of skill you're adding (design-and-build vs. review workflow); they're the actual spec.

If your skill has a slash-command launcher, add one file under `plugins/staff-engineer-skills/commands/`, and add it to the catalog in `commands/skills.md` and the tables in `README.md`.

## Adding a new plugin

This repo currently ships one plugin (`staff-engineer-skills`), covering backend reliability, review, and frontend craft. If a contribution is a genuinely different kind of thing (not a fit for any of those categories), it can get its own plugin: create `plugins/<plugin-name>/` with its own `.claude-plugin/plugin.json`, `skills/`, and (optionally) `commands/`/`agents/`, then add an entry to the `plugins` array in `.claude-plugin/marketplace.json`. Bar for this is high; prefer adding to the existing plugin's categories first.

## Review process for skills

Before a new or substantially changed skill is merged, it should go through:

1. **Adversarial review**: the three-persona pass (Saboteur / New Hire / Security Auditor) that every skill in this repo has already been through. Findings get fixed, not argued around.
2. **Domain-expert review** where relevant (e.g., a security-adjacent skill gets a security-focused pass).

This isn't a formality. It's how every existing skill here reached its current state, and it's caught real bugs (an incorrect composition-order claim, a negative-cache sentinel returned as real data, a spoofable rate-limit key). Expect the same rigor on new contributions.

## Small fixes

Typos, broken links, clarifications: just open a PR. No process overhead for these.
