# gitwork

[简体中文](README.md) | [English](README.en.md)

A Git finishing skill for Codex. After a project-based development task is implemented and verified, it commits only the changes produced by that task while protecting the user's existing staged and uncommitted work.

## What it does

`gitwork` is a mandatory Git finishing gate for every implementation task in the current user-selected workspace folder. It does not decide whether the folder already “looks like a project”: regardless of technology stack, project type, size, existing contents, or Git status, it should trigger whenever a task is expected to create, modify, rename, or delete files there, even when the user never mentions Git or commits. It applies both to new projects in empty or non-Git folders and to existing projects; frontend pages, backend services, full-stack apps, CLIs, libraries, scripts, demos, and prototypes are non-exhaustive examples only. It will:

- Confirm the project root explicitly selected in Codex instead of guessing from the terminal path.
- Initialize Git with `main` when the selected project has no repository, after reviewing the existing files for safety.
- Record the Git state before work begins and preserve existing staged or uncommitted changes.
- Stage and commit only the changes produced by the current task after implementation and verification.
- Write commit subjects as `<type>: <Chinese description>`, such as `fix: 修复登录状态丢失问题`.
- Skip the automatic commit and explain why when the task changes cannot be isolated safely.

It does not run for Q&A, planning, reviews, tasks with no file changes, or chats without an explicitly selected project folder. It also does not push, rewrite history, bypass Git hooks, or mix unrelated changes into a commit.

## Installation

### Install with one AI prompt (recommended)

Open this repository as a project in Codex, then paste this sentence directly into the chat:

```text
Install the gitwork skill from the current project: copy ./gitwork to ~/.codex/skills/gitwork, verify that SKILL.md and agents/openai.yaml are intact, and remind me to restart Codex when finished.
```

You can also ask the AI to install directly from GitHub without downloading the repository first:

```text
Use skill-installer to install gitwork from https://github.com/Niall-Young/Gitwork/tree/main/gitwork, then remind me to restart Codex.
```

### Manual installation

Run these commands from the repository root:

```bash
mkdir -p ~/.codex/skills
cp -R ./gitwork ~/.codex/skills/gitwork
```

If you use a custom `CODEX_HOME`, change the destination to `$CODEX_HOME/skills/gitwork`.

Restart Codex after installation so it can discover the new skill.

## Usage

Implicit invocation is enabled. Once installed, use Codex normally for a development task that changes files in a selected project; the skill will perform the Git finishing workflow when its conditions are met.

You can also invoke it explicitly:

```text
Use $gitwork to complete this project task and commit only the changes produced by this task.
```

When finished, Codex reports the commit hash and subject, the verification performed, and any uncommitted changes that remain in the repository.

## Workflow

1. Confirm the user-selected project root and check for an in-progress Git operation.
2. Record the branch, index, unstaged changes, and untracked files as the task baseline.
3. Complete the requested development work and verify it appropriately.
4. Compare against the baseline and select only task-owned files or hunks.
5. Inspect the staged content and diff, then create one conventionally formatted commit.
6. Verify `HEAD` and the worktree state before reporting the result.

## Safety boundaries

- Task commits never use `git add .` or `git add -A`.
- Secrets, local environment files, dependency trees, caches, build artifacts, and unexpectedly large binaries are not committed.
- Routine task commits are skipped during an unresolved merge, rebase, cherry-pick, revert, or detached HEAD state.
- The skill does not override Git identity, use `--no-verify`, amend commits, rewrite history, or push automatically.
- If task-owned changes cannot be separated reliably from existing user changes, the worktree is preserved and the conflict is reported.

## Project layout

```text
.
├── gitwork/
│   ├── SKILL.md             # Trigger conditions, Git safety rules, and workflow
│   └── agents/openai.yaml   # Codex metadata and default prompt
├── AGENTS.md                # Development and validation guidance
├── README.md                # Chinese documentation
└── README.en.md             # English documentation
```
