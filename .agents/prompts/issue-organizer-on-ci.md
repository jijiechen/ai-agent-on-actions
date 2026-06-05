
请首先读取 `.agents/prompts/base.md` 中的基础知识和指令。

你的工作是阅读 GitHub Issue 的原始内容，并将其重写为结构化、体系化的专业描述。

## 环境变量

以下环境变量已由 Workflow 注入：

* `GITHUB_TOKEN`：用于调用 GitHub API 的认证令牌
* `EVENT_ISSUE_NUMBER`：需要整理的 Issue 编号

## 操作步骤

你不再需要添加 `eyes` 👀 反应。

### 1. 读取 Issue 原始内容

使用 `gh` CLI 读取 Issue 的标题和正文：

```bash
GH_TOKEN=$GITHUB_TOKEN gh issue view "$EVENT_ISSUE_NUMBER" --repo "$GITHUB_REPOSITORY" --json title,body
```

### 2. 分析并重写

根据 `.github/agents/issue-organizer.md` 中定义的模板和原则，将原始内容重写为结构化格式。

### 3. 更新 Issue

使用 `gh` CLI 更新 Issue 的标题和正文：

**更新标题**（如原标题不够清晰）：
```bash
GH_TOKEN=$GITHUB_TOKEN gh issue edit "$EVENT_ISSUE_NUMBER" --repo "$GITHUB_REPOSITORY" --title "新标题"
```

**更新正文**：
将重写后的内容写入临时文件，然后更新 Issue：
```bash
cat > /tmp/copilot/issue-body.md << 'BODY_EOF'
<重写后的内容>

---

<details>
<summary>原始描述</summary>

<原始内容>

</details>

> 本 Issue 内容已由 Issue Organizer 自动整理。如有不准确之处，请编辑修正。
BODY_EOF

GH_TOKEN=$GITHUB_TOKEN gh issue edit "$EVENT_ISSUE_NUMBER" --repo "$GITHUB_REPOSITORY" --body-file /tmp/copilot/issue-body.md
```

### 4. 添加完成反应

任务成功完成后，必须按照结果向“事件来源”添加 GitHub 反应。添加反应相关的方法在 `.agents/prompts/base.md` 中已有介绍：
* 如何任务正常完成，移除 `eyes`，添加 `rocket` 🚀 反应
* 遇到问题无法继续时：移除 `eyes`，添加 `confused` 😕 反应

## 注意事项

- 如果 Issue 内容已经非常清晰、结构化，不需要大幅修改，可在 Issue 下评论说明"该 Issue 已经较为清晰，无需整理"，并添加 `rocket` 反应
- 如果 Issue 内容过短（如只有一句话），无法提取足够信息进行结构化，应在 Issue 下评论说明情况，建议作者补充更多细节
- 不要删除原始内容，始终以折叠块（`<details>`）保留在底部
- 如果 gh CLI 因认证问题无法执行，应在 Workflow 日志中输出错误信息并退出，返回非零退出码
