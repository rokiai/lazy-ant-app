# 懒蚂蚁 MCP Server

懒蚂蚁提供本地 stdio MCP Server，供 Cursor、Codex、WorkBuddy 等客户端读写 Markdown 草稿、主题和「我的模板」Skill。完整模块与 Tool 契约见 [MCP Server README](../apps/desktop/src/mcp-server/README.md)。

## 启动与配置

### 安装包用户

在懒蚂蚁首页打开客户端教程，使用「复制 MCP 配置」。启动器位于：

- macOS：`/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp`
- Windows：安装目录 `resources\lazyant-mcp.cmd`
- Linux：安装目录 `resources/lazyant-mcp`

启动器通过 `ELECTRON_RUN_AS_NODE` 运行 `Resources/mcp/stdio.js`，并与桌面端共用 userData。

Cursor 配置示例：

```json
{
  "mcpServers": {
    "懒蚂蚁": {
      "command": "/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp",
      "args": [],
      "env": {
        "LAZYANT_DATA_DIR": "/Users/you/Library/Application Support/懒蚂蚁"
      }
    }
  }
}
```

Codex 配置示例：

```toml
[mcp_servers."懒蚂蚁"]
command = "/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp"
args = []

[mcp_servers."懒蚂蚁".env]
LAZYANT_DATA_DIR = "/Users/you/Library/Application Support/懒蚂蚁"
```

也可使用：

```bash
codex mcp add "懒蚂蚁" -- /Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp
```

### 源码开发

```bash
LAZYANT_DATA_DIR="$HOME/Library/Application Support/懒蚂蚁" \
  pnpm --filter @lazy-ant/desktop cli:mcp
```

Cursor 开发配置：

```json
{
  "mcpServers": {
    "懒蚂蚁": {
      "command": "pnpm",
      "args": ["--dir", "/path/to/lazy-ant/apps/desktop", "cli:mcp"],
      "env": {
        "LAZYANT_DATA_DIR": "/Users/you/Library/Application Support/懒蚂蚁"
      }
    }
  }
}
```

构建安装包时，`build:app` 会调用 `build:mcp`；`electron-builder` 将 `out/mcp/` 和启动器复制到 `Resources/`。stdio 的 stdout 只承载协议数据，不能输出调试日志。

## 通用工作流

所有客户端都按 Tool / Resource 工作：先读取对应 `get_*_spec` 或 Resource，再按用户意图调用 create 或 update。客户端未展示 MCP Prompt 时，不影响这条流程。

支持 MCP Prompt 的客户端可额外展示以下快捷入口；它们会在模型生成前注入对应规范，但不是 MCP 的唯一工作路径：

| Prompt                 | 落盘 Tool                                              |
| ---------------------- | ------------------------------------------------------ |
| `write_article`        | `create_article_draft` 或 `update_article_draft`       |
| `write_image_text`     | `create_image_text_draft` 或 `update_image_text_draft` |
| `create_article_theme` | `create_article_theme` 或 `update_article_theme`       |
| `create_image_theme`   | `create_image_theme` 或 `update_image_theme`           |

用户在 `/write_article`、`/write_image_text` 等文字后同一条消息中写的要求、素材和正文，都属于本次需求；文字本身不是可自动执行的 Tool 命令。客户端未提供 Prompt，或未传 `brief` 时，Agent 直接使用同条用户消息作为要求，读取 `get_*_spec` 后继续执行，不应中断。

可读取以下 Resource，效果等同对应 `get_*_spec` Tool：

| Resource                             | 说明               |
| ------------------------------------ | ------------------ |
| `lazyant://spec/article-markdown`    | 文章 Markdown 语法 |
| `lazyant://spec/image-text-markdown` | 图文 Markdown 语法 |
| `lazyant://spec/article-theme`       | 文章主题 CSS 规范  |
| `lazyant://spec/image-theme`         | 图文主题 CSS 规范  |

## 多篇草稿

- 用户要求写一篇、再写一篇或换选题时，调用 `create_*_draft`。每次都返回新的 `draftId`，同一会话可以有多篇。
- 用户要求修改、润色、续写或基于现有文章时，已知 `draftId` 就 `get_*_draft` 后携带 `revision` 调用 `update_*_draft`。
- 只有目标 `draftId` 未知时才 `search_*_drafts`；必须提供 `query`，每页最多 20 篇。
- `draftId` 是唯一文章身份。MCP 不用会话、`taskId`、当前稿或 `article-ready.json` 推断目标。
- MCP 草稿只使用 Markdown `content`，不会读取或写入 `contentHtml`、主题色、主题引用或发布自动化数据。

`revision` 是乐观并发控制版本。另一个 Agent 或编辑器先完成写入时，旧 revision 会被拒绝；应重新读取目标草稿后再更新。

## Skill 选择与管理

先确定内容：链接先读取完整正文，粘贴素材先读完，只有选题时先明确受众和核心要点。内容确定前，不能因为“公众号”“财经”等词提前选择 Skill。

内容确定后，使用 `search_workspace_skills(query)` 搜索「我的模板」轻量摘要，再以 `get_workspace_skill(skillId)` 读取选中包。自动写作不会读取市场 Skill 或 `~/.agents/skills`；没有合适模板时直接按 Markdown 规范写作。

模板写入使用显式 create/update/delete：

| Tool                     | 规则                                                           |
| ------------------------ | -------------------------------------------------------------- |
| `create_workspace_skill` | 新建包；必须一次性提供包含 `SKILL.md` 的 `files`，不覆盖同名包 |
| `update_workspace_skill` | 先 get，再传整包 `revision`、`path` 和 `content` 或 `edits`    |
| `delete_workspace_skill` | 仅在用户明确要求删除时调用，必须带最新整包 `revision`          |

`skill-packs` 是权威目录。`~/.agents/skills` 只是启动及模板变更后的镜像，由同步服务对齐和清理；它不参与 MCP 搜索、读取或更新。

## 主题管理

主题工具采用 `search → get → update`：

1. `themeRef` 未知时调用 `search_article_themes` 或 `search_image_themes`，提供关键词。
2. `get_article_theme` 或 `get_image_theme` 返回 `themeRef`、完整主题和 `revision`。
3. 调用对应 `update_*_theme`，传回 `themeRef + revision`。局部 CSS 使用 `themeCSSEdits`，整体重写使用 `themeCSS`。

`themeRef` 是内部定位符，模型从搜索结果取得，不应要求用户提供。服务内部处理市场/个人来源；市场目录只读时会报错，不会隐式创建个人副本。只有用户明确新建或再做一套时调用 `create_article_theme` 或 `create_image_theme`。

写文章和图文时不得读取主题，主题皮和主题色由用户在编辑器中选择。

## Tool 列表

当前公开 25 个 Tool：

| 分类     | Tool                                                                                                                           |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 规范     | `get_article_markdown_spec`、`get_image_text_markdown_spec`、`get_article_theme_spec`、`get_image_theme_spec`                  |
| 文章     | `search_article_drafts`、`get_article_draft`、`create_article_draft`、`update_article_draft`                                   |
| 图文     | `search_image_text_drafts`、`get_image_text_draft`、`create_image_text_draft`、`update_image_text_draft`                       |
| 我的模板 | `search_workspace_skills`、`get_workspace_skill`、`create_workspace_skill`、`update_workspace_skill`、`delete_workspace_skill` |
| 文章主题 | `search_article_themes`、`get_article_theme`、`create_article_theme`、`update_article_theme`                                   |
| 图文主题 | `search_image_themes`、`get_image_theme`、`create_image_theme`、`update_image_theme`                                           |

成功结果同时提供 JSON 文本和 `structuredContent`；未知 Tool 是 MCP `MethodNotFound`。产品规则失败、版本冲突和只读资源等可恢复错误以 Tool `isError` 返回。

## 数据边界

| 产物          | 路径（相对 userData）                      |
| ------------- | ------------------------------------------ |
| 文章/图文定稿 | `workspace/articles/drafts/<draftId>.json` |
| 个人文章主题  | `workspace/themes/article/<folderId>/`     |
| 个人图文主题  | `workspace/themes/image/<folderId>/`       |
| 我的模板      | `skill-packs/<skillId>/`                   |

MCP 不处理账号、Cookie、平台发布、浏览器调试端口或自动任务。自动任务独立维护自己的状态，不能影响 MCP 的草稿选择和更新。

## 验证

```bash
pnpm --dir apps/desktop run build:mcp
pnpm --dir apps/desktop exec vitest run src/mcp-server
```
