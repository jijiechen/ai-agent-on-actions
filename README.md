# 项目介绍

本项目是一个基于 GitHub Actions 运行的自动化 AI Coding Agent。它能够监听仓库中的 Issue 和评论事件，自动调用大语言模型（LLM）执行代码开发、测试验证、需求整理等任务。基本实现类似于 GitHub Copilot 云端 Agent 的能力。

要求对接的 LLM API 支持 OpenAI chat-completions 格式接口。

# 快速开始

> **注意**：作为本仓库的维护者，在首次使用前需将可复用 Workflow 部署到 `.github/workflows/` 目录。详见文末 [仓库管理员部署说明](#仓库管理员部署说明)。

作为最终用户，你**无需复制本仓库的全部内容**。只需在你的目标仓库中进行以下两步操作：

## 第一步：配置 Secrets 和 Variables

在你的目标仓库 `Settings > Secrets and variables > Actions` 中添加：

| 类型 | 名称 | 是否必填 | 说明 | 默认值 |
|------|------|---------|------|--------|
| Secret | `LLM_API_KEY` | **必填** | 调用大语言模型的 API Key | - |
| Variable | `LLM_API_BASE_URL` | 可选 | LLM API 基准地址 | `https://api.deepseek.com` |
| Variable | `LLM_MODEL_GENERIC` | 可选 | 用于常规任务的模型 | `deepseek-v4-flash` |
| Variable | `LLM_MODEL_EXPERT` | 可选 | 用于开发/测试任务的模型 | `deepseek-v4-flash` |

## 第二步：在目标仓库中添加 Workflow 文件

在你的目标仓库中创建 `.github/workflows/ai-agent.yml`，内容如下：

```yaml
name: AI Agent

on:
  issues:
    types: [labeled]
  issue_comment:
    types: [created]

jobs:
  run-ai-agent:
    if: >
      (github.event_name == 'issues' && github.event.label.name == 'ai-agent')
      ||
      (github.event_name == 'issue_comment' && startsWith(github.event.comment.body, '/ai-agent '))

    uses: jijiechen/ai-agent-on-actions/.github/workflows/ai-agent.yml@main
    permissions:
      contents: write
      issues: write
      pull-requests: write
    secrets:
      LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
    # 若要自定义 LLM 配置，取消以下注释：
    # with:
    #   llm_api_base_url: ${{ vars.LLM_API_BASE_URL }}
    #   llm_model_generic: ${{ vars.LLM_MODEL_GENERIC }}
    #   llm_model_expert: ${{ vars.LLM_MODEL_EXPERT }}
```

> 你也可以直接复制仓库中的 `example-workflows/auto-agent.yml` 文件到你的 `.github/workflows/` 目录。

## 前提条件

在你的目标仓库的 `Settings > Actions > General` 中启用：
* **Allow GitHub Actions to create and approve pull requests**（允许 GitHub Actions 创建和批准 Pull Request）

# 触发 AI Agent

### 方式一：为 Issue 添加 `ai-agent` 标签

1. 在仓库中创建一个 Issue，描述你的需求（例如：新增功能、修复问题等）
2. 为该 Issue 添加 `ai-agent` 标签
3. GitHub Actions 将自动运行，AI Agent 会分析 Issue 内容并执行相应任务

### 方式二：在 Issue 或 PR 中发送指令

1. 在任意 Issue 或 Pull Request 的评论中，以 `/ai-agent ` 开头编写你的指令
2. 例如：`/ai-agent 请帮我修复登录页面的样式问题`
3. AI Agent 会读取评论内容并开始工作

# 可用 Agent 类型

触发后，系统会先通过通用 Agent 分析任务类型，然后自动分派到对应的专业 Agent：

| Agent | 用途 |
|-------|------|
| **dev** | 开发 Agent，负责实现新功能或修复已发现的问题，编写自动化测试 |
| **tester** | 测试 Agent，负责执行黑盒功能测试，验证业务实现的正确性或修复的有效性 |
| **issue-organizer** | 需求整理 Agent，负责将混乱的 Issue 描述重写为清晰、结构化的内容 |

## AI Agent 能做什么

- 根据 Issue 描述自动编写或修改代码
- 针对代码变更自动运行测试并验证结果
- 自动创建 Pull Request 并推送代码
- 将不清晰的 Issue 整理为结构化的开发需求
- 在 Issue 和 PR 上留言反馈工作进展和结果

# 高级用法

## Composite Action（直接调用指定 Agent）

如果你需要在现有 Workflow 的某个步骤中直接调用特定 Agent（而非自动路由），可以使用 Composite Action：

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: jijiechen/ai-agent-on-actions@main
    env:
      COPILOT_PROVIDER_API_KEY: ${{ secrets.LLM_API_KEY }}
    with:
      agent: dev  # dev / tester / issue-organizer / generic
```

完整示例见 `example-workflows/direct-agent.yml`。

## 版本锁定

建议在生产环境中锁定到特定版本标签，而非使用 `@main`：

```yaml
uses: jijiechen/ai-agent-on-actions/.github/workflows/ai-agent.yml@v1.0.0
```

并使用 `agent_ref` 参数确保 Prompt 文件版本一致：

```yaml
with:
  agent_ref: 'v1.0.0'
```

## 代理配置

如果你的运行环境需要通过 HTTP 代理访问 LLM API，可在调用方 Workflow 中设置环境变量：

```yaml
jobs:
  run-ai-agent:
    uses: jijiechen/ai-agent-on-actions/.github/workflows/ai-agent.yml@main
    env:
      HTTP_PROXY: ${{ vars.HTTP_PROXY }}
      HTTPS_PROXY: ${{ vars.HTTPS_PROXY }}
```

# 仓库管理员部署说明

> 以下内容仅供本仓库（`jijiechen/ai-agent-on-actions`）管理员参考。最终用户无需关心。

## 可复用 Workflow 部署

由于 GitHub Actions 运行环境的 `GITHUB_TOKEN` 默认缺少 `workflows` 写入权限，可复用 Workflow 文件无法由 CI 自动部署到 `.github/workflows/` 目录。

请仓库管理员手动执行：

```bash
cp reusable-workflow.yml .github/workflows/ai-agent.yml
git add .github/workflows/ai-agent.yml
git commit -m "chore: 部署可复用 Workflow（workflow_call 支持）"
git push
```

部署完成后，外部仓库即可通过以下方式引用：

```yaml
uses: jijiechen/ai-agent-on-actions/.github/workflows/ai-agent.yml@main
```

### 后续自动化部署

部署完成后，建议在 `.github/workflows/ai-agent.yml` 的 `permissions` 块中添加 `workflows: write`，以便后续 CI 运行时可自动更新 Workflow 文件：

```yaml
permissions:
  contents: write
  issues: write
  pull-requests: write
  workflows: write  # 新增此行
```

