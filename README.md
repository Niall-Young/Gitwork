# gitwork

[简体中文](README.md) | [English](README.en.md)

一个为 Codex 准备的 Git 收尾 skill。它会在项目型开发任务完成并验证后，只提交本次任务产生的改动，同时尽力保护用户原有的暂存内容和未提交工作。

## 它有什么用

`gitwork` 是当前用户所选工作区文件夹内所有落盘实现任务的强制 Git 收尾门禁。它不判断文件夹是否已经“像个项目”；无论技术栈、项目类型、规模、现有内容或 Git 状态，只要任务预计会在该文件夹内创建、修改、重命名或删除文件，即使用户没有提到 Git 或提交也应触发。它既适用于空文件夹或非 Git 文件夹里的新项目，也适用于已有项目；前端页面、后端服务、全栈应用、CLI、库、脚本、demo 和原型都只是非穷举示例。它会：

- 确认 Codex 明确选中的项目根目录，不根据当前终端路径猜测项目。
- 在项目尚未使用 Git 时，以 `main` 为默认分支初始化仓库，并安全检查已有文件。
- 记录任务开始前的 Git 状态，避开用户原有的暂存或未提交改动。
- 在任务完成并验证后，只暂存和提交本次任务的变更。
- 使用 `<type>: <中文说明>` 格式生成提交信息，例如 `fix: 修复登录状态丢失问题`。
- 在无法安全隔离改动时放弃自动提交，并明确说明原因。

它不会用于纯问答、规划、评审、没有文件改动的任务，或未明确绑定项目文件夹的聊天。它也不会自动推送、改写历史、跳过 Git hooks，或把无关改动混入提交。

## 安装

### 让 AI 一句话安装（推荐）

在 Codex 中打开本仓库作为项目，然后把下面这句话直接复制给 AI：

```text
请从当前项目安装 gitwork skill：将 ./gitwork 复制到 ~/.codex/skills/gitwork，确认 SKILL.md 和 agents/openai.yaml 完整，并在完成后提醒我重启 Codex。
```

也可以直接让 AI 从 GitHub 安装，无需先下载仓库：

```text
请使用 skill-installer 从 https://github.com/Niall-Young/Gitwork/tree/main/gitwork 安装 gitwork，并在完成后提醒我重启 Codex。
```

### 手动安装

在仓库根目录运行：

```bash
mkdir -p ~/.codex/skills
cp -R ./gitwork ~/.codex/skills/gitwork
```

如果你设置了自定义 `CODEX_HOME`，请将目标目录改为 `$CODEX_HOME/skills/gitwork`。

安装完成后重启 Codex，以便加载新 skill。

## 使用方式

该 skill 已允许隐式调用。安装后，正常让 Codex 在已选项目中完成会产生文件改动的开发任务即可；符合条件时，它会自动执行 Git 收尾。

也可以显式指定：

```text
使用 $gitwork 完成这个项目任务，并只提交本次任务产生的变更。
```

完成后，Codex 会报告提交哈希、提交信息、执行过的验证，以及仓库中仍然存在的未提交改动。

## 工作流程

1. 确认用户明确选择的项目根目录，并检查 Git 操作状态。
2. 记录分支、暂存区、未暂存和未跟踪文件作为任务基线。
3. 完成用户要求的开发工作并进行相应验证。
4. 对比基线，只选择本次任务产生的文件或代码块。
5. 检查暂存内容和 diff，再创建一个符合约定格式的提交。
6. 复核 `HEAD` 与工作区状态后报告结果。

## 安全边界

- 任务提交禁止使用 `git add .` 或 `git add -A`。
- 不会提交密钥、环境文件、依赖目录、缓存、构建产物或异常的大型二进制文件。
- 不会在 merge、rebase、cherry-pick、revert 未解决或 detached HEAD 状态下创建常规任务提交。
- 不会覆盖 Git 身份、使用 `--no-verify`、修改已有提交或自动推送。
- 如果本次改动无法与用户已有改动可靠分离，会保留工作区原状并说明情况。

## 项目结构

```text
.
├── gitwork/
│   ├── SKILL.md             # 触发条件、Git 安全规则和完整工作流
│   └── agents/openai.yaml   # Codex 展示信息与默认提示词
├── AGENTS.md                # 项目开发与验证约定
├── README.md                # 中文说明
└── README.en.md             # English documentation
```
