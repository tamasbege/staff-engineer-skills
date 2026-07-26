# staff-engineer-skills

Claude Code plugin marketplace. The repo is the marketplace: `.claude-plugin/marketplace.json` lists the plugins, each plugin lives in `plugins/<name>/` with its own `.claude-plugin/plugin.json`, `skills/<skill-name>/SKILL.md`, and `commands/*.md`.

## Versioning — the rule that matters most

**Installed users only receive updates when the plugin's own version changes.** Claude Code keys update detection off `plugins/<name>/.claude-plugin/plugin.json` → `version`, not off git commits. A content change without a version bump is invisible to everyone who already installed the plugin.

Therefore, on **every** change:

1. **Any change inside `plugins/<name>/`** (skill text, command, manifest description — anything) → bump that plugin's `version` in its `plugin.json`. Patch for fixes/wording, minor for new skills/commands, major for renames or removals of skills users may reference.
2. **Any noteworthy repo change** (including the above, plus docs like README/CONTRIBUTING) → bump `metadata.version` in `.claude-plugin/marketplace.json` and add a CHANGELOG entry describing it.
3. Never commit a plugin content change without its version bump in the same commit. This has been missed once already (see CHANGELOG 1.1.2) — treat it as part of the change, not an afterthought.

Plugins version independently: adding or fixing one plugin does not touch the others' versions. The marketplace version is the human-facing release number the CHANGELOG narrates; plugin versions are the mechanically meaningful ones.

## Release checklist (any content change)

1. Make the change.
2. Bump the affected plugin's `version` (and only that plugin's).
3. Bump `metadata.version` in `marketplace.json`.
4. Add a CHANGELOG entry under the new marketplace version.
5. If skills/commands were added, renamed, or removed: update the README tables and the plugin's discovery command catalog (e.g. `commands/hard-parts.md`).
6. Commit (normal commit, no history rewriting — the repo is public) and push. Pushing to `main` is the release.

## Conventions

- **Skills must be standalone.** Never reference skills outside this repo (they exist only in the author's environment and become dangling references for installers). Same-plugin sibling references are fine — they travel together on install.
- **House style for skills** is defined in `CONTRIBUTING.md`: trigger-rich third-person `description` frontmatter, Phase 0 output-format question (HTML default / Markdown), mandatory context gathering with codebase-first inspection and a partial-context protocol, concrete reference examples, anti-patterns table, testable constraints, deliverables checklist. Read an existing SKILL.md before writing a new one.
- **Review before merge:** new or substantially changed skills go through adversarial review (and a domain-expert pass where relevant) before commit; fixes get recorded in the CHANGELOG. See `CONTRIBUTING.md`.
- **Derived work gets credit.** If a skill adapts ideas from an external source, name the source and license in the skill file (see `blind-spot-breaker` for the pattern).
- Git identity for commits: `tamasbege <begetamas@gmail.com>`. Line endings: repo files are LF; CRLF warnings from git on Windows are expected noise.
