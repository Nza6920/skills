# Repository Guidelines

## Project Structure & Module Organization

This repository publishes reusable Agent Skills, not the CLI implementations they invoke.

- `skills/<skill-name>/SKILL.md` is the entry point for each published skill.
- `README.md` and `README.zh-CN.md` document the collection in English and Chinese; keep their facts and links aligned.
- `db-cli/` and `loki-query/` are ignored, independent Git worktrees used to develop the corresponding CLIs. Commit CLI changes in those repositories, not here.
- `.agents/` and `skills-lock.json` are local tooling inputs and are ignored.

There are currently no repository-level source, test, or asset directories. Keep skill-specific references or scripts beside their `SKILL.md` when they become necessary.

## Build, Test, and Development Commands

This Markdown-only repository has no build step. Use lightweight checks before submitting changes:

```bash
rg --files skills -g 'SKILL.md'  # list published skill entry points
git diff --check                 # detect whitespace errors
git status --short               # verify the intended change set
```

Test a changed skill by invoking it in a representative prompt and confirming that every branch stops or completes at its stated criterion. CLI implementation tests belong in the linked CLI repository.

## Writing Style & Naming Conventions

Use kebab-case for skill directories, and make the frontmatter `name` exactly match the directory name. Each `SKILL.md` must begin with valid YAML frontmatter containing a concise, trigger-oriented `description`. Retain `disable-model-invocation: true` for skills that access databases, production logs, or other explicitly selected environments.

Write instructions in direct English, organize procedures as numbered steps, and give each step a checkable completion criterion. Prefer links to CLI documentation over duplicating flags that may become stale. Wrap Markdown prose near the existing style and use fenced blocks for commands or prompt examples.

## Testing Guidelines

Review every modified path and invocation branch. Confirm that examples name a profile where required, sensitive values remain in environment variables, and read-only boundaries are explicit. When README content changes, update both language versions in the same change.

## Commit & Pull Request Guidelines

The repository has no commit history yet, so no established message convention exists. Use short, imperative subjects such as `Add db-query skill` or `Clarify Loki approval boundary`.

Pull requests should explain the behavior changed, list validation performed, and link the associated CLI change when relevant. Include screenshots only for rendered Markdown issues; otherwise provide a concise before/after example.
