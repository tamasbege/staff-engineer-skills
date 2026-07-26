---
description: Run the four-persona adversarial code review (launches adversarial-code-reviewer)
---

Invoke the `adversarial-code-reviewer` skill now and follow it exactly: gather the changes (unstaged+staged diff by default, or whatever the user specified below), run all four personas end to end, each must surface at least one finding, then deduplicate/promote and output the structured verdict.

Arguments (may specify `--diff <ref>`, `--file <path>`, a PR number, or be empty for the default diff): $ARGUMENTS
