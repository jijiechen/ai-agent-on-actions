# 项目介绍

本项目是一个空的模板项目，用于演示基于 GitHub Actions 运行基于 Copilot Cli 的自动化 AI Coding Agent。基本实现类似于 GitHub Copilot 云端 Agent 的能力。

要求你对接的 LLM API 支持 OpenAI chat-completions 格式接口。

# 配置步骤

在你的组织、项目里启用 GitHub Actions 并启用下列功能：
* Allow GitHub Actions to create and approve pull requests

在项目仓库里添加以下 Actions 变量（Variables）：
* `LLM_MODEL_GENERIC`: 可选，用于处理常规任务、需求整理的模型名称，如 `deepseek-v4-flash` （默认为 `deepseek-v4-flash`）
* `LLM_MODEL_EXPERT`: 可选，用于处理开发和测试任务的模型名称，如 `deepseek-v4-pro`（默认为 `deepseek-v4-flash`）
* `LLM_API_BASE_URL`: 可选，你的 LLM API 的 API 基准地址（默认为 `https://api.deepseek.com`）

在项目仓库里添加以下 Actions 密钥（Secrets）：
* `LLM_API_KEY`: 用于调用大语言模型的 API Key

