# GitHub Agentic Workflows 设计规格书

**日期**: 2026-07-01  
**状态**: 设计中  
**仓库**: test-gaw（五子棋人机对战游戏）

---

## 1. 概述

为仓库配置基于 Claude Code CLI 的自动化工作流，实现 Issue → AI 分类 → AI 开发 → AI 自审 → 提 PR → 人类合并的完整链路。

## 2. 架构总览

```
人类提 Issue → [WF1: 分类] → feat/bug → [WF2: 开发+审查+PR] → 人类审查合并
                                    ↓
                              invalid / needs-info → 关闭/追问
```

三个核心组件：
- **WF1** (`issue-classify.yml`)：Issue 分类
- **WF2** (`ai-dev-review.yml`)：AI 开发 + 自审 + PR
- **规则文件** (`classify.md`, `review.md`)：可人工调整的 prompt 规则

## 3. 技术选型

| 项目 | 选择 |
|---|---|
| AI 后端 | Claude Code CLI |
| CI 平台 | GitHub Actions (ubuntu-latest) |
| Git 操作 | `gh` CLI（Actions 自带）|
| Secrets | `ANTHROPIC_BASE_URL`, `ANTHROPIC_AUTH_TOKEN` |

## 4. Workflow 1：Issue 分类

### 触发条件
```yaml
on:
  issues:
    types: [opened]
```

### 流程

1. 检出仓库代码
2. 安装 Claude Code CLI
3. 读取 `.github/ai-workflows/classify.md` 作为分类规则
4. Claude Code 分析 issue 内容，输出分类结果
5. 根据分类执行动作

### 分类及动作

| 分类 | 标签 | 动作 |
|---|---|---|
| `invalid`（无意义） | `invalid` | 关闭 issue，回复说明原因 |
| `needs-info`（待补充） | `needs-info` | 回复追问具体信息 |
| `feat`（功能开发） | `feat` | 回复确认，触发 WF2 |
| `bug`（缺陷修复） | `bug` | 回复确认，触发 WF2 |

## 5. Workflow 2：AI 开发 + 自审 + PR

### 触发条件
```yaml
on:
  issues:
    types: [labeled]
```
过滤 `feat` 和 `bug` 标签。

### 流程

#### 5.1 开发阶段
1. 读取 issue 描述 + 现有代码
2. 创建分支 `ai/<issue-number>-<slug>`
3. 在分支上开发（保守模式，优先保持现有架构）
4. 提交代码（commit message 关联 issue）

#### 5.2 自审阶段
1. Claude Code 以审查者视角检查 diff
2. 按 `.github/ai-workflows/review.md` 规则逐项检查
3. 输出：通过 / 不通过 + 问题列表

#### 5.3 修复循环（最多 3 轮）
- 不通过 → 按问题列表修复 → 重新自审
- 3 轮后仍不通过 → 在 issue 中报告失败原因，标记 `needs-human`，停止

#### 5.4 提 PR
- 审查通过 → `gh pr create`
- PR 标题关联 issue（如 `Fix #12: 修复棋盘渲染 Bug`）
- PR 描述含：改动摘要、审查结果、关联 issue
- 添加 `ai-generated` 标签

## 6. 文件结构

```
.github/
├── workflows/
│   ├── issue-classify.yml      # WF1: Issue 分类
│   └── ai-dev-review.yml       # WF2: AI 开发+审查+PR
├── ai-workflows/
│   ├── classify.md             # 分类规则 prompt
│   └── review.md               # 审查规则 prompt
```

无需额外依赖，全部使用 GitHub Actions 原生能力 + Claude Code CLI + `gh` CLI。

## 7. Secrets 配置

| Secret | 说明 |
|---|---|
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `ANTHROPIC_AUTH_TOKEN` | 自定义认证 token |

`GH_TOKEN` 由 Actions 自动提供，无需手动配置。

## 8. 关键约束

- **开发范围**：保守模式，优先保持单文件架构，仅必要时拆分或引入依赖
- **代码风格**：遵循现有代码风格，不做无关重构
- **安全性**：Claude Code CLI 无法获取 secrets 之外的环境变量
- **并发**：同一时间仅允许一个 WF2 运行（使用 concurrency 限制），避免分支冲突
