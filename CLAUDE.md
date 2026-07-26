# staff-engineer-skills

Claude Code plugin marketplace. The repo is the marketplace: `.claude-plugin/marketplace.json` lists the plugins, each plugin lives in `plugins/<name>/` with its own `.claude-plugin/plugin.json`, `skills/<skill-name>/SKILL.md`, and `commands/*.md`.

## Versioning: the rule that matters most

**Installed users only receive updates when the plugin's own version changes.** Claude Code keys update detection off `plugins/<name>/.claude-plugin/plugin.json` → `version`, not off git commits. A content change without a version bump is invisible to everyone who already installed the plugin.

Therefore, on **every** change:

1. **Any change inside `plugins/<name>/`** (skill text, command, manifest description, anything) → bump that plugin's `version` in its `plugin.json`. Patch for fixes/wording, minor for new skills/commands, major for renames or removals of skills users may reference.
2. **Any noteworthy repo change** (including the above, plus user-facing docs like README/CONTRIBUTING) → bump `metadata.version` in `.claude-plugin/marketplace.json` and add a CHANGELOG entry describing it. Exception: internal contributor docs (this file) don't need a release entry, since the rule targets changes users can see or install.
3. Never commit a plugin content change without its version bump in the same commit. This has been missed once already (see CHANGELOG 1.1.2), so treat it as part of the change, not an afterthought.

Plugins version independently: adding or fixing one plugin does not touch the others' versions. The marketplace version is the human-facing release number the CHANGELOG narrates; plugin versions are the mechanically meaningful ones.

## Release checklist (any content change)

1. Make the change.
2. Bump the affected plugin's `version` (and only that plugin's).
3. Bump `metadata.version` in `marketplace.json`.
4. Add a CHANGELOG entry under the new marketplace version.
5. If skills/commands were added, renamed, or removed: update the README tables and the discovery command catalog (`commands/skills.md`).
6. Commit (normal commit, no history rewriting since the repo is public) and push. Pushing to `main` is the release.
7. Tag the commit `v<marketplace-version>` (e.g. `v1.1.3`) and push the tag, then create/update the matching GitHub Release with notes (the first release covered the whole history to date; subsequent ones can scope notes to what changed since the last tag, linking `CHANGELOG.md` for full detail). This is discoverability only (installs/updates read `marketplace.json`/`plugin.json` directly, not tags or releases), but do it every time so it doesn't silently fall behind. `gh` in this environment is not authenticated to the repo's actual GitHub host; create the tag with plain git, then either paste notes into `https://github.com/tamasbege/staff-engineer-skills/releases/new?tag=<tag>` or ask the user to auth `gh` to that host first.

## Renames touch many files

Renaming a skill (or plugin) fans out further than it looks. Grep the old name across the whole repo and expect hits in: the skill directory name, `SKILL.md` frontmatter `name` + H1, any `commands/*.md` that invoke it, the plugin's `plugin.json` description, the root `marketplace.json` description, the README tables, and the plugin's discovery-command catalog. Leave historical CHANGELOG entries under the old name; they're an accurate record of what that release was called.

## Verifying changes

There is no CI. Before committing, at minimum:

- Parse every touched manifest: `python -c "import json;json.load(open('...'))"` (or equivalent) on `marketplace.json` and any `plugin.json`.
- Check each new/renamed skill: directory name matches frontmatter `name`; frontmatter has `name` + `description`; the description is third-person and trigger-rich (it drives auto-invocation).
- Grep for dangling references (old names, external skills) across `plugins/`, README, and command files.
- To smoke-test locally, the marketplace can be added from the working copy and plugins reloaded with `/reload-plugins` in a Claude Code session.

## Environment quirks

- The `gh` CLI here is authenticated against a GitHub **Enterprise** host, not github.com, so `gh repo view`/`gh api` against `tamasbege/staff-engineer-skills` fail even though `git push` over https works fine. Don't burn time on it; use plain git, and give the user copy-paste steps for anything requiring the GitHub web UI or a github.com-authenticated `gh`.

## Conventions

- **Skills must be standalone.** Never reference skills outside this repo (they exist only in the author's environment and become dangling references for installers). Same-plugin sibling references are fine; they travel together on install.
- **House style for skills** is defined in `CONTRIBUTING.md`: trigger-rich third-person `description` frontmatter, mandatory context gathering with codebase-first inspection and a partial-context protocol, concrete reference examples, anti-patterns table, testable constraints, deliverables checklist. Design-and-build skills additionally ask a Phase 0 output-format question (HTML default / Markdown); fast review-workflow skills (`blindspot-finder`, `ux-reviewer`) skip that and output a structured verdict inline instead. Read an existing SKILL.md before writing a new one, picking one matching the kind of skill you're adding.
- **Review before merge:** new or substantially changed skills go through adversarial review (and a domain-expert pass where relevant) before commit; fixes get recorded in the CHANGELOG. See `CONTRIBUTING.md`.
- **Derived work gets credit.** If a skill adapts ideas from an external source, name the source and license in the skill file (see `blindspot-finder` for the pattern).
- Git identity for commits: `tamasbege <begetamas@gmail.com>`. Line endings: repo files are LF; CRLF warnings from git on Windows are expected noise.
