# GitHub Agentic Workflows 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为五子棋游戏仓库配置基于 Claude Code CLI 的自动化 Issue→开发→审查→PR 工作流

**Architecture:** 两个 GitHub Actions Workflow + 两个 Prompt 规则文件。WF1 在 issue 创建时触发分类；WF2 在 feat/bug 标签添加时触发 AI 开发+自审+PR

**Tech Stack:** GitHub Actions, Claude Code CLI (npm), `gh` CLI, `jq`

## Global Constraints

- Claude Code CLI 通过 `@anthropic-ai/claude-code` npm 包安装
- API 凭证使用 `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN` 两个 secrets
- 分支命名：`ai/<issue-number>-<slug>`
- 自审最多 3 轮，失败标记 `needs-human`
- 保守开发模式：优先保持单文件架构
- WF2 使用 concurrency 限制，同一时间仅允许一个实例运行

---

### Task 1: 创建目录结构和分类规则文件

**Files:**
- Create: `.github/ai-workflows/classify.md`

- [ ] Step 1: 创建目录结构 `mkdir -p .github/workflows .github/ai-workflows`
- [ ] Step 2: 写入 classify.md（分类规则 prompt，定义四种分类标准及 JSON 输出格式）
- [ ] Step 3: 提交

### Task 2: 创建审查规则文件

**Files:**
- Create: `.github/ai-workflows/review.md`

- [ ] Step 1: 写入 review.md（审查规则 prompt，四个审查维度）
- [ ] Step 2: 提交

### Task 3: 创建 WF1 — Issue 分类 Workflow

**Files:**
- Create: `.github/workflows/issue-classify.yml`

- [ ] Step 1: 写入 issue-classify.yml
- [ ] Step 2: 提交

### Task 4: 创建 WF2 — AI 开发+自审+PR Workflow

**Files:**
- Create: `.github/workflows/ai-dev-review.yml`

- [ ] Step 1: 写入 ai-dev-review.yml
- [ ] Step 2: 提交
