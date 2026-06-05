# 项目介绍

本项目是一个基于 GitHub Actions 运行的自动化 AI Coding Agent 模板。它能够监听仓库中的 Issue 和评论事件，自动调用大语言模型（LLM）执行代码开发、测试验证、需求整理等任务。基本实现类似于 GitHub Copilot 云端 Agent 的能力。

要求对接的 LLM API 支持 OpenAI chat-completions 格式接口。

# 使用方式

作为最终用户，你无需在本地安装任何工具。只需在 GitHub 仓库中完成以下操作即可触发 AI Agent 自动工作。

## 触发 AI Agent

### 方式一：为 Issue 添加 `ai-agent` 标签

1. 在仓库中创建一个 Issue，描述你的需求（例如：新增功能、修复问题等）
2. 为该 Issue 添加 `ai-agent` 标签
3. GitHub Actions 将自动运行，AI Agent 会分析 Issue 内容并执行相应任务

### 方式二：在 Issue 或 PR 中发送指令

1. 在任意 Issue 或 Pull Request 的评论中，以 `/ai-agent ` 开头编写你的指令
2. 例如：`/ai-agent 请帮我修复登录页面的样式问题`
3. AI Agent 会读取评论内容并开始工作

## 可用 Agent 类型

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

# 配置步骤（仓库管理员）

## 前提条件

在你的组织或仓库中启用 GitHub Actions，并在仓库设置中启用以下功能：
* **Allow GitHub Actions to create and approve pull requests**（允许 GitHub Actions 创建和批准 Pull Request）

## Actions 变量（Variables）

在仓库的 `Settings > Secrets and variables > Actions > Variables` 中添加以下变量：

| 变量名 | 是否必填 | 说明 | 默认值 |
|--------|---------|------|--------|
| `LLM_MODEL_GENERIC` | 可选 | 用于常规任务和需求整理的模型名称 | `deepseek-v4-flash` |
| `LLM_MODEL_EXPERT` | 可选 | 用于开发和测试任务的模型名称 | `deepseek-v4-flash` |
| `LLM_API_BASE_URL` | 可选 | LLM API 的基准地址 | `https://api.deepseek.com` |

## Actions 密钥（Secrets）

在仓库的 `Settings > Secrets and variables > Actions > Secrets` 中添加以下密钥：

| 密钥名 | 是否必填 | 说明 |
|--------|---------|------|
| `LLM_API_KEY` | **必填** | 调用大语言模型的 API Key |

## 从模板创建

你可以通过 GitHub 的 "Use this template" 功能将此仓库作为模板创建新仓库，上述配置在新仓库中同样需要完成。

