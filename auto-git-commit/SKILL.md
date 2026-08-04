---
name: auto-git-commit
description: "在 Codex 中为绑定了用户所选项目文件夹的软件开发任务自动完成 Git 收尾：任务实际创建、修改、重命名或删除项目文件并验证后，在项目根目录初始化 Git（缺失时使用 main），只提交本次任务变更，并使用 feat、fix、chore 等类型加中文说明。用于所有带项目文件夹且会产生文件改动的代码任务；不要用于未绑定文件夹的最近聊天、纯问答、规划、评审或没有项目文件变化的任务。"
---

# Codex Project Auto Git Commit

Run this workflow around every qualifying task. Treat safe ownership of the commit as more important than forcing a commit.

## Gate Git operations

Perform this gate before running any Git command:

1. Confirm that the Codex runtime explicitly supplies a user-selected project/workspace folder. Do not infer a project from the shell working directory, code in the conversation, or generated temporary files.
2. Ignore Codex-managed internal locations such as `~/.codex/visualizations`. If there is no unambiguous user-selected project folder, stop this workflow without running `git init`, `git add`, or `git commit`.
3. Use the selected folder itself as the only project root. Do not inherit a Git repository above it, search for repositories below it, or operate on paths outside it.
4. Continue only when the task will create, modify, rename, or delete files inside that root. Skip read-only, explanation, research, planning, and review tasks.

## Establish the baseline

Run this at the start of the task, before editing files:

1. Resolve the canonical project-root path.
2. Recognize an existing repository only when `<project-root>/.git` exists as a directory or file. When it exists, verify that `git -C <project-root> rev-parse --show-toplevel` resolves to the same canonical root. Stop and report an unsafe mismatch.
3. For an existing repository, record the current branch, operation state, `git status --short`, staged paths, unstaged paths, untracked paths, and relevant diffs. Do not alter the user's index while recording the baseline.
4. Do not create a routine task commit during an unresolved merge, rebase, cherry-pick, revert, or detached-HEAD state. Let a task explicitly handling that Git operation own its completion; otherwise report the condition.
5. When the project root has no `.git`, initialize exactly there with `git init -b main`. Never initialize a parent or child directory.

For a newly initialized repository with existing files:

1. Inspect the complete file list and create or minimally extend `.gitignore` for the detected stack. Exclude local environment files, credentials and private keys, dependency directories, caches, logs, generated output, and build artifacts. Preserve deliberate example files such as `.env.example`.
2. Review every candidate before staging. Do not stage secrets, unexpectedly large binaries, generated dependencies, or files outside the project root.
3. Stage the reviewed baseline. This is the only phase where `git add -A` is allowed, and only after the safety review and ignore rules are in place.
4. Inspect the staged names and diff, then commit `chore: 初始化 Git 仓库` when committable files exist. Do not create an empty baseline commit.
5. If Git identity, hooks, or another Git error prevents the baseline commit, do not bypass it or invent identity values. Continue the requested work when safe, but report that automatic committing is blocked.

## Finish and isolate task changes

Complete the requested implementation and its proportionate verification before committing.

1. Compare the final state with the recorded baseline. If the task produced no project-file changes, do not create a commit.
2. Select only files and hunks produced by the current task. Exclude unrelated user changes and generated or ignored artifacts. Do not use `git add .` or `git add -A` for a task commit.
3. When a task touches a file that was already dirty, stage only clearly separable task hunks. If any required task change cannot be separated reliably, do not create an incomplete or mixed commit; leave the task changes uncommitted and explain the conflict.
4. Preserve unrelated staged entries. When unrelated content is already staged and the task paths are wholly task-owned, use a path-limited commit such as `git commit --only ... -- <task-paths>` so the existing index content is not included.
5. Inspect task-scoped staged names, `git diff --cached --check`, the diff stat, and the actual diff. Confirm that the commit contains the complete task and nothing else.

## Write the commit message

Create one task commit using exactly `<type>: <中文说明>`. Omit scope and trailing punctuation. Choose the type that best represents the primary user-visible outcome:

- `feat`: add capability or behavior
- `fix`: correct a defect
- `refactor`: restructure code without changing behavior
- `perf`: improve performance
- `test`: change tests only
- `docs`: change documentation only
- `build`: change dependencies or build tooling
- `ci`: change continuous-integration configuration
- `style`: change formatting only
- `chore`: perform maintenance not covered above
- `revert`: revert an earlier change

Keep the Chinese subject concise and specific, for example `feat: 添加批量导出功能` or `fix: 修复登录状态丢失问题`.

Run normal Git hooks. Never use `--no-verify`, create an empty commit, amend an existing commit, rewrite history, or push unless the user explicitly asks. If a hook fails, fix only in-scope failures, re-stage the task changes, and retry; otherwise leave the changes uncommitted and report the failure.

## Report the result

After a successful commit:

1. Read the short commit hash and subject from `HEAD`.
2. Run `git status --short` and distinguish remaining user changes from task leftovers.
3. Report the hash, commit subject, verification performed, and any remaining uncommitted changes.

If no commit was created, state the exact reason. Never claim success from `git commit` output without verifying `HEAD`.
