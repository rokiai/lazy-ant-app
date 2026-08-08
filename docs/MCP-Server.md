# 懒蚂蚁 MCP Server

本文档是懒蚂蚁 MCP Server 的唯一维护入口，覆盖客户端接入、MCP v2 契约、Tool 工作流、用户数据边界、主题快照格式和开发态官方资源同步。懒蚂蚁提供本地 stdio MCP Server，供 Cursor、Codex、WorkBuddy 等客户端读写 Markdown 草稿、主题和「我的模板」Skill。完整模块与 Tool builder 职责见 [MCP Server README](../apps/desktop/src/mcp-server/README.md)。

## 文档定位与维护原则

- MCP 只写入用户个人数据：Markdown 草稿、个人主题和「我的模板」Skill；市场资源仅作为只读内容提供。
- 市场主题与市场 Skill 是只读官方资源；MCP 不直接修改市场源码目录。市场主题包必须包含与 `theme.json`、`theme.css` 匹配的 `.mcp-revision`，开发态写回也通过该 marker 发布提交。
- 过时协议、旧字段和错误入口直接删除，不保留 migration、fallback、双读双写或兼容别名。
- 修改 Tool 的输入、输出、持久化或资源边界时，必须同步更新本文档、实现和测试。

## 启动与配置

### 安装包用户

在懒蚂蚁首页打开客户端教程。macOS / Windows 支持「一键配置」写入全局文件；Linux 暂不支持一键写入，但仍可「复制 MCP 配置」后手动写入。启动器位于：

- macOS：`/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp`
- Windows：安装目录 `resources\lazyant-mcp.cmd`
- Linux：安装目录 `resources/lazyant-mcp`（仅复制配置，需手动确认路径）

启动器通过 `ELECTRON_RUN_AS_NODE` 运行 `Resources/mcp/stdio.js`，并与桌面端共用 userData。

Cursor 配置示例：

```json
{
  "mcpServers": {
    "lazyant": {
      "type": "stdio",
      "command": "/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp",
      "args": [],
      "env": {
        "LAZYANT_DATA_DIR": "/Users/you/Library/Application Support/懒蚂蚁"
      }
    }
  }
}
```

Cursor 全局配置文件为 `~/.cursor/mcp.json`；WorkBuddy 全局配置文件为 `~/.codebuddy/.mcp.json`。一键配置只增量替换 `mcpServers.lazyant`，不会覆盖其他服务，也不写项目级配置。

Codex 配置示例：

```toml
[mcp_servers.lazyant]
command = "/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp"
args = []

[mcp_servers.lazyant.env]
LAZYANT_DATA_DIR = "/Users/you/Library/Application Support/懒蚂蚁"
```

也可使用：

```bash
codex mcp add lazyant -- /Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp
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
    "lazyant": {
      "type": "stdio",
      "command": "pnpm",
      "args": ["--dir", "/path/to/lazy-ant/apps/desktop", "cli:mcp"],
      "env": {
        "LAZYANT_DATA_DIR": "/Users/you/Library/Application Support/懒蚂蚁"
      }
    }
  }
}
```

构建安装包时，`build:app` 会调用 `build:mcp`；`electron-builder` 将 `out/mcp/` 和启动器复制到 `Resources/`。MCP 服务版本直接读取桌面端 `package.json`，与桌面包版本保持一致。stdio 的 stdout 只承载协议数据，不能输出调试日志；传输和实例错误通过 `onerror` 写入 stderr。

## MCP 协议边界

- stdio 使用 `serveStdio(..., { legacy: 'serve' })`，现代客户端使用带 `_meta` 的 2026-07-28 opening；仍使用 `2025-11-25 initialize` 的客户端会由 SDK 固定到独立的 2025-era 连接。
- Tool 注册公开稳定的 `inputSchema` 和 `outputSchema`；调用参数缺失、类型错误和未知 Tool 按 MCP 协议返回错误。
- 成功结果同时提供 JSON 文本和 `structuredContent`，结构化内容不得出现 output schema 未声明的字段。
- 产品规则失败、版本冲突和只读资源等可恢复错误以 `isError: true` 的 Tool 结果返回。
- 每个 Tool 都公开安全语义：规范、search、get 为只读且幂等；create、update 为非只读、非破坏且非幂等；delete 为破坏性且幂等。客户端应根据 `readOnlyHint`、`destructiveHint` 和 `idempotentHint` 展示相应操作风险。
- Prompt 和 Resource 是规范入口，不替代 Tool 的权限和持久化边界。

## 通用工作流

所有客户端都按 Tool / Resource 工作：先读取对应 `get_*_spec` 或 Resource，再按用户意图调用 create 或 update。客户端未展示 MCP Prompt 时，不影响这条流程。

支持 MCP Prompt 的客户端可额外展示以下快捷入口；它们会在模型生成前注入对应规范，但不是 MCP 的唯一工作路径：

| Prompt                | 落盘 Tool                                              |
| --------------------- | ------------------------------------------------------ |
| `write_article`       | `create_article_draft` 或 `update_article_draft`       |
| `write_image_text`    | `create_image_text_draft` 或 `update_image_text_draft` |
| `write_article_theme` | `create_article_theme` 或 `update_article_theme`       |
| `write_image_theme`   | `create_image_theme` 或 `update_image_theme`           |

用户在 `/write_article`、`/write_image_text` 等文字后同一条消息中写的要求、素材和正文，都属于本次需求；文字本身不是可自动执行的 Tool 命令。Prompt 的 `brief` 参数是必填的；客户端未提供 Prompt 时，Agent 直接使用同条用户消息作为要求，读取 `get_*_spec` 后继续执行。

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
- 旧草稿缺少 `generatedAt` 时，MCP 返回 `generatedAt: null`，以满足当前 output schema。

`revision` 是乐观并发控制版本。另一个 Agent 或编辑器先完成写入时，旧 revision 会被拒绝；应重新读取目标草稿后再更新。

## Skill 选择与管理

先确定内容：链接先读取完整正文，粘贴素材先读完，只有选题时先明确受众和核心要点。内容确定前，不能因为“公众号”“财经”等词提前选择 Skill。

内容确定后，使用 `search_workspace_skills(query)` 搜索「我的模板」轻量摘要，再以 `get_workspace_skill(skillId)` 读取选中包。自动写作不会读取市场 Skill 或 `~/.agents/skills`；没有合适模板时直接按 Markdown 规范写作。

模板写入使用显式 create/update/delete：

| Tool                     | 规则                                                                     |
| ------------------------ | ------------------------------------------------------------------------ |
| `create_workspace_skill` | 新建包；必须一次性提供包含 `SKILL.md` 的 `files`，不覆盖同名包           |
| `update_workspace_skill` | 先 get，再传整包 `revision`、`path` 和 `content` 或 `edits`              |
| `delete_workspace_skill` | 仅在用户明确要求删除时调用，必须带最新整包 `revision` 和 `confirm: true` |

`skill-packs` 是权威目录。MCP 只读写该目录，不直接访问外部 Skill 镜像。macOS / Windows 下桌面应用启动时执行一次对齐，并将「我的 Skill」同步到 Codex、Cursor 共用的 `~/.agents/skills` 以及 WorkBuddy 的 `~/.codebuddy/skills`；Linux 暂不自动落盘，不创建外部镜像目录。运行期间由 `skill-packs` watcher 在模板变更后调度同步。

桌面单文件编辑、整包覆盖和 MCP 更新共用同一份整包内容 `revision` 校验和跨进程写锁；市场安装只负责准备工作区目录，不提供隐式 market 回退写入。`search_workspace_skills` 与桌面一级列表使用的 `scanRevision` 仅用于轻量变更检测，不能作为任何写入请求的 `revision`。

镜像同步使用台账记录所有权，只会替换或删除台账中明确由懒蚂蚁创建的目录。未托管目录不会被删除；目标目录与待同步模板同名但没有懒蚂蚁台账记录时保留原目录，并在同步结果的 `conflicts` 中报告冲突。台账损坏、版本不合法或目标根不匹配时同步会停止，不会把台账当作空记录继续清理。同步锁校验持锁进程和 token，避免长时间同步被误判为过期。

Skill 包的可编辑内容有统一边界：最多 64 个 `.md` / `.json` 文件，单文件最多 256 KiB，全部可编辑内容最多 2 MiB（按 UTF-8 字节计算）。搜索摘要、读取、创建、更新和市场 Skill 安装均执行这些限制；超限直接返回可恢复错误，不截断或静默丢文件。

## 主题管理

主题工具采用 `search → get → update`：

1. `themeRef` 未知时调用 `search_article_themes` 或 `search_image_themes`，提供关键词。
2. `get_article_theme` 或 `get_image_theme` 返回 `themeRef`、完整主题和 `revision`。
3. 调用对应 `update_*_theme`，传回 `themeRef + revision`。局部 CSS 使用 `themeCSSEdits`，整体重写使用 `themeCSS`。

`themeRef` 是内部定位符，模型从搜索结果取得，不应要求用户提供。主题来源和写入边界如下：

| 引用                  | 读取 | MCP 更新                    | 物理写入                          |
| --------------------- | ---- | --------------------------- | --------------------------------- |
| `market:<folderId>`   | 允许 | 拒绝，提示先收藏            | MCP 禁止写入 `market/`            |
| `personal:<folderId>` | 允许 | 允许，必须带最新 `revision` | 只写 `userData/workspace/themes/` |

MCP 只会把主题创建到「我的主题」对应的个人主题目录；`market:<folderId>` 更新会返回可恢复错误。用户收藏市场主题后，编辑器会复制市场主题的当前 JSON、CSS 和必要资源，形成独立个人副本；之后修改只写个人副本，不再与市场主题关联。只有用户明确新建或再做一套时调用 `create_article_theme` 或 `create_image_theme`。

编辑器新建主题、收藏市场主题、保存主题和取消收藏都通过同一套 workspace 主题写入服务落盘；`workspace/themes` 是唯一权威，IndexedDB 只保存扫描后的渲染缓存。编辑器保存会携带当前快照 `revision`，与 MCP 并发修改时遵守同样的乐观并发校验；磁盘写入成功后才刷新缓存。

市场与个人主题包都以 `theme.json + theme.css + .mcp-revision` 作为一个快照提交：两个文件写完后才发布提交标记，读取会校验标记与内容一致，避免并发时读到混合版本。CSS 只持久化在独立的 `theme.css`，不从 `theme.json` 的旧内嵌字段回退读取。市场主题运行时只读，开发态同步也必须通过同一提交契约。

个人可写主题包的文件契约：

```text
theme.json       元数据、颜色、布局和运行配置；不保存 article.themeCSS/image.themeCSS 文本
theme.css        唯一 CSS 来源
.mcp-revision    theme.json + theme.css 的提交版本
```

个人包读取时 marker、JSON、CSS 任一缺失或不一致都视为无效快照；不回读 JSON 内嵌 CSS，不接受 marker 缺失的隐式兼容格式。市场只读包同样不回读 JSON 内嵌 CSS，摘要和详情必须从同一份快照派生。

开发态编辑器另有“同步官方资源到 market”入口：

- 主题实现：`apps/desktop/src/main/market/theme-pack.ts`，写回 `apps/desktop/market/themes/{article|image}`。
- Skill 实现：`apps/desktop/src/main/market/skill-pack/...`，写回 `apps/desktop/market/skills`。

该 IPC 流程与 MCP Tool 更新完全分离，只允许开发态使用；正式版禁止自动写回源码目录。审核后的官方资源才通过此流程进入市场。

这里的“市场文章主题”“市场图文主题”和“市场 Skill 模板”在生成阶段都先通过 MCP 创建到用户的个人目录（主题位于 `workspace/themes`，Skill 位于 `skill-packs`）。只有开发态编辑器的官方资源同步流程，才会把审核后的主题写回 `apps/desktop/market/themes`；MCP 不直接写市场源码目录，正式版也不提供该写入能力。

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

成功结果同时提供 JSON 文本和 `structuredContent`，每个 Tool 都公开稳定的 `outputSchema`；MCP v2 `registerTool` 会按 `tools/list` 的 `inputSchema` 和 `outputSchema` 校验调用边界，缺参或类型错误以 `isError` Tool 结果返回。未知 Tool 是 MCP `MethodNotFound`。产品规则失败、版本冲突和只读资源等可恢复错误也以 Tool `isError` 返回。

## 数据边界

| 产物          | 路径（相对 userData）                      |
| ------------- | ------------------------------------------ |
| 文章/图文定稿 | `workspace/articles/drafts/<draftId>.json` |
| 个人文章主题  | `workspace/themes/article/<folderId>/`     |
| 个人图文主题  | `workspace/themes/image/<folderId>/`       |
| 我的模板      | `skill-packs/<skillId>/`                   |

MCP 不处理账号、Cookie、平台发布、浏览器调试端口或自动任务。自动任务独立维护自己的状态，不能影响 MCP 的草稿选择和更新。

## 验证

文档或 MCP 契约变更后，至少运行：

```bash
pnpm exec vitest run
pnpm run typecheck:node
pnpm run typecheck:web
pnpm exec tsc --noEmit -p tsconfig.mcp.json --composite false
pnpm run build:mcp
pnpm run verify:mcp
git diff --check
```

stdio 验证应覆盖：带 2026-07-28 `_meta` 契约的现代 opening、`2025-11-25 initialize` 握手、`tools/list`、`prompts/list`、`resources/list` 和关键 Tool 调用成功；`market:` 更新返回 `isError` 且不修改市场文件。

修改 MCP 实现或打包产物后，须重新 `build:mcp` 并在客户端重启 MCP 连接，避免客户端继续枚举旧 Tool。

本文是 MCP Server 的唯一长期维护入口；实现或协议变更应直接更新本文和对应测试。
