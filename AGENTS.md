# AGENTS.md

## Project overview

This repository contains `auto-git-commit`, a Codex skill that safely finishes project-based coding tasks with an isolated Git commit. The skill is intentionally conservative: it must protect existing user changes, avoid operating outside the explicitly selected project root, and prefer skipping a commit over creating an unsafe or mixed one.

## Repository layout

- `auto-git-commit/SKILL.md` is the source of truth for trigger conditions, Git safety rules, the commit workflow, and reporting requirements.
- `auto-git-commit/agents/openai.yaml` contains the user-facing skill metadata and enables implicit invocation.

Keep the package small. Do not add scripts, dependencies, examples, or extra documentation unless they make the skill materially safer or easier to maintain.

## Editing guidelines

- Treat Git safety and preservation of user work as the highest-priority behavior.
- Keep the distinction between the Codex-selected project root and an inferred shell or repository path explicit.
- Never weaken the rules against staging unrelated changes, secrets, generated dependencies, or unexpectedly large binaries.
- Preserve existing staged entries and dirty files. If task-owned changes cannot be isolated reliably, the documented behavior must be to leave them uncommitted and explain why.
- Do not introduce history-rewriting, hook-bypassing, pushing, or broad staging behavior. In particular, task commits must not use `git add .` or `git add -A`.
- Keep commit subjects in the documented form `<type>: <中文说明>`, with a conventional type and a concise Chinese description.
- Write normative workflow instructions in clear imperative English. Keep user-facing metadata and commit-message examples in Chinese.
- When trigger behavior changes, update both the `description` frontmatter in `SKILL.md` and `agents/openai.yaml` so they remain consistent.

## Validation

There is no automated test suite. Before finishing a change:

1. Confirm `SKILL.md` still has valid YAML frontmatter with the exact skill name `auto-git-commit`.
2. Confirm `agents/openai.yaml` is valid YAML and its prompt references `$auto-git-commit`.
3. Review the workflow against at least these cases: a clean existing repository, a dirty repository with unrelated changes, a newly initialized repository, an unresolved Git operation, and a task that creates no files.
4. Check that no instruction can stage or commit files outside the selected project root.
5. Run `git diff --check` and inspect the complete diff.

## Scope discipline

Make the smallest change that satisfies the request. Do not modify the skill merely to restyle prose. If a requested change would trade away isolation or user-data safety, call out the conflict instead of silently implementing it.
