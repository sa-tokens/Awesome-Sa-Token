---
name: git-commit-push
description: 'Git 提交并推送工作流。检查当前改动，自动生成 commit message，列出提交信息、推送地址和分支，等待用户确认后再执行 commit 和 push。支持多个远程仓库全量推送。触发关键词：提交代码、git commit、git push、推送代码、提交改动。'
argument-hint: '可选：传入 1/ok/yes/确认 可跳过确认步骤直接提交；或附加提交说明，如 "feat: 新增登录功能"'
---

# Git 提交与推送工作流

## 适用场景

- 需要提交并推送当前工作区改动
- 不确定 commit message 怎么写，让 AI 自动生成
- 仓库配置了多个远程推送地址，需要全量推送

## 执行步骤

### 第 1 步：检查当前改动

运行以下命令，了解当前状态：

```bash
git status
git diff --stat
```

如果没有任何改动，告知用户并终止流程。

### 第 2 步：生成 commit message

分析改动内容（`git diff`），按 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/) 规范生成提交信息：

```
<type>(<scope>): <subject>

[可选 body]
```

常用 type：`feat` / `fix` / `docs` / `refactor` / `chore` / `style` / `test`

### 第 3 步：收集推送信息

运行以下命令获取远程仓库和分支配置：

```bash
git remote -v
git branch --show-current
git config --get branch.<current>.remote
git config --get branch.<current>.merge
git diff --numstat           # 获取未暂存文件的增删行数
git diff --cached --numstat  # 获取已暂存文件的增删行数
git status --short           # 获取每个文件的状态（M/A/D/?? 等）
```

整理出：
- 所有配置的推送地址（去重 `push` 类型的 remote）
- 当前分支名
- 每个 remote 对应的目标分支（如无特殊配置，使用当前分支名）

### 第 4 步：向用户展示确认清单

**在执行任何 git 操作前**，先检查调用时是否携带了确认参数：

- 若参数包含 `1` / `ok` / `yes` / `确认`（如 `/git-commit-push 1`），展示清单后**直接进入第 6 步**，无需等待用户输入
- 否则，展示清单后进入第 5 步等待确认

以结构化方式列出以下信息：

```
📋 提交确认清单

📝 Commit Message：
  <生成的提交信息>

📦 改动文件（共 N 个）：
  状态  增删行数        文件路径
  ----  -----------    ------------------------------------------
  M     +12 / -3       src/main/java/com/example/Foo.java
  A     +56 / -0       src/main/java/com/example/Bar.java
  D     +0  / -20      src/main/java/com/example/Old.java

🚀 推送计划：
  Remote    地址                      分支
  --------  ------------------------  ------
  origin    https://github.com/...   main
  mirror    https://gitee.com/...    main

输入 1 / ok / yes / 确认  → 开始提交并推送
输入 0 / no / 取消        → 放弃本次操作
```

### 第 5 步：等待用户确认

| 用户输入 | 含义 | 处理方式 |
|----------|------|----------|
| `1` / `ok` / `yes` / `确认` | 确认无误，开始工作 | 继续执行第 6 步 |
| `0` / `no` / `取消` | 放弃本次操作 | 终止流程，不做任何改动 |
| 其他文字 | 要求修改 | 根据反馈修改 commit message 或推送目标，重新展示清单，再次等待确认 |

### 第 6 步：执行提交与推送

```bash
git add -A
git commit -m "<commit message>"
```

对每个 remote 依次推送：

```bash
git push <remote> <本地分支>:<目标分支>
```

推送完成后汇报每个 remote 的推送结果（成功 / 失败及错误原因）。

## 注意事项

- **绝不跳过确认步骤**：即使用户说"帮我提交一下"，也必须先展示确认清单
- 如果 `git push` 提示需要设置上游分支，使用 `--set-upstream` 参数
- 如果存在推送冲突，告知用户并给出建议（pull rebase 或 force push），**不自动执行破坏性操作**
- 对于 `--force` / `--force-with-lease` 等操作，必须额外单独询问用户确认
