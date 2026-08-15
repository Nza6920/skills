# Repository Guidelines

## Project Structure & Sources of Truth

This repository coordinates independent Agent Skill and CLI projects. It does
not publish duplicate root-level skill copies.

- `db-cli/.agents/skills/db-query/SKILL.md` is the source of truth for
  `$db-query`.
- `loki-query/.agents/skills/loki-query/SKILL.md` is the source of truth for
  `$loki-query`.
- Keep the root `skills/` directory absent. Change a skill only in its owning
  subproject.
- Keep `README.md` and `README.zh-CN.md` factually and semantically aligned.
- `db-cli/` and `loki-query/` are ignored, independent Git worktrees. Commit
  their CLI, skill, and project documentation changes inside those repositories.
- Root `.agents/` and `skills-lock.json` are ignored local tooling inputs, not
  published skill sources.

There are no root source, test, or asset directories. The root repository owns
only workspace guidance and the project index.

## Validation Commands

The root repository has no build step. Validate the workspace boundary and the
changed repository explicitly:

```bash
test -f db-cli/.agents/skills/db-query/SKILL.md
test -f loki-query/.agents/skills/loki-query/SKILL.md
test ! -d skills
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

Commit each change in the repository that owns it. Stage exact paths so an
independent worktree or unrelated local tooling is not included accidentally.
Use short imperative subjects such as `Clarify database approval boundary` or
`Document Loki query limits`.

Pull requests should explain the behavior changed and validation performed.
Link related changes across the root index and a subproject when both are
updated.
