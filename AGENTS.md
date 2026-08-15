# Repository Guidelines

## Project Structure & Sources of Truth

This repository distributes Agent Skills and coordinates their independent CLI
projects. The root `skills/` directory contains published copies synchronized
from the owning subprojects.

- `db-cli/.agents/skills/db-query/SKILL.md` is the source of truth for
  `$db-query`.
- `loki-query/.agents/skills/loki-query/SKILL.md` is the source of truth for
  `$loki-query`.
- `skills/db-query/` and `skills/loki-query/` are the corresponding published
  copies. Change a skill in its owning subproject first, then synchronize the
  complete directory into `skills/`.
- Keep `README.md` and `README.zh-CN.md` factually and semantically aligned.
- `db-cli/` and `loki-query/` are ignored, independent Git worktrees. Commit
  their CLI, skill, and project documentation changes inside those repositories.
- Root `.agents/` and `skills-lock.json` are ignored local tooling inputs, not
  published skill sources.

There are no root source, test, or asset directories. The root repository owns
workspace guidance, the project index, and the published skill copies.

## Validation Commands

The root repository has no build step. Validate the workspace boundary and the
changed repository explicitly:

```bash
test -f db-cli/.agents/skills/db-query/SKILL.md
test -f loki-query/.agents/skills/loki-query/SKILL.md
test -f skills/db-query/SKILL.md
test -f skills/db-query/agents/openai.yaml
test -f skills/loki-query/SKILL.md
test -f skills/loki-query/agents/openai.yaml
cmp db-cli/.agents/skills/db-query/SKILL.md skills/db-query/SKILL.md
cmp db-cli/.agents/skills/db-query/agents/openai.yaml skills/db-query/agents/openai.yaml
cmp loki-query/.agents/skills/loki-query/SKILL.md skills/loki-query/SKILL.md
cmp loki-query/.agents/skills/loki-query/agents/openai.yaml skills/loki-query/agents/openai.yaml
git diff --check
git status --short
git -C db-cli status --short
git -C loki-query status --short
```

Run a subproject's documented tests when its CLI changes. For a skill-only
change, inspect every invocation branch and confirm that each step has a
checkable completion criterion.

## Writing Conventions

Within each subproject, use kebab-case skill directories and match frontmatter
`name` to the directory name. Retain `disable-model-invocation: true` and
`allow_implicit_invocation: false` for skills that access databases, production
logs, or other explicitly selected environments.

Write skill procedures in direct English as numbered steps with checkable
completion criteria. Prefer links to the owning CLI documentation over copied
syntax that can become stale. Keep credentials in environment variables and
make read-only and approval boundaries explicit.

When project README content changes, update both language versions in the same
subproject commit. Root README changes must also keep both languages aligned.

## Commit & Pull Request Guidelines

Commit source changes in the repository that owns them and synchronized
published copies in this root repository. Stage exact paths so an independent
worktree or unrelated local tooling is not included accidentally. Use short
imperative subjects such as `Clarify database approval boundary` or `Document
Loki query limits`.

Pull requests should explain the behavior changed and validation performed.
Link related changes across the root index and a subproject when both are
updated.
