# 懒蚂蚁 MCP Server

懒蚂蚁对外提供 MCP Server（stdio），供 [Cursor](https://cursor.com)、[OpenAI Codex](https://developers.openai.com/codex)、Claude Desktop 等客户端连接，读写文章、图文、Skill 与主题。

## 启动方式

### 开发 / 源码仓库

在仓库根目录（开发专用，需本机已装 pnpm）：

```bash
LAZYANT_DATA_DIR="$HOME/Library/Application Support/懒蚂蚁" \
  pnpm --filter @lazy-ant/desktop cli:mcp
```

### 正式安装包（DMG / 安装程序）

安装懒蚂蚁后无需 Node / pnpm。在首页点击客户端打开教程，再点 **「复制 MCP 配置」**：

- **Cursor / WorkBuddy**：复制 JSON，粘贴到对应 MCP 设置
- **Codex**：复制 TOML，追加到 `~/.codex/config.toml`

Cursor 示例：

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

启动器通过 `ELECTRON_RUN_AS_NODE` 运行打包在 `Resources/mcp/stdio.js` 的 MCP Server，与桌面端共用同一份 `userData` 与市场主题资源。

Windows 对应 `Resources\lazyant-mcp.cmd`；Linux 对应 `resources/lazyant-mcp`。

### 打包与 CI

发版流水线**会一起打 MCP**进安装包，无需单独步骤：

1. `pnpm run build:app` → 末尾执行 `build:mcp`（`vite` 打出 `out/mcp/stdio.js`）
2. `electron-builder` 通过 `extraResources` 复制：
   - `out/mcp/` → `Resources/mcp/`
   - `build/lazyant-mcp`（及 `.cmd`）→ `Resources/lazyant-mcp`

日常 CI（`.github/workflows/ci.yml`）也会跑 `build:mcp`，避免 MCP 打包回归拖到发版才发现。Release workflow 的 `build:app` 同样包含上述步骤。

## 在 Cursor 中配置

1. 打开懒蚂蚁首页 → 点击 **Cursor** → **复制 MCP 配置**
2. 粘贴到 Cursor MCP 设置或项目 `.cursor/mcp.json`
3. 重启 Cursor，确认「懒蚂蚁」已连接

### 开发态（源码仓库）

若从源码开发，可在 Cursor **Settings → MCP** 或项目 `.cursor/mcp.json` 中添加：

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

**正式用户请优先从首页打开对应客户端教程后复制配置**（安装包指向 `lazyant-mcp` 启动器），不要手抄开发态的 `pnpm` 命令。

## 在 OpenAI Codex 中配置

Codex（CLI / IDE 扩展）通过 **TOML** 注册 MCP，配置文件为 `~/.codex/config.toml` 或项目 `.codex/config.toml`。

### 安装包用户（推荐）

1. 懒蚂蚁首页 → 点击 **Codex** → **复制 MCP 配置**
2. 将剪贴板内容追加到 `~/.codex/config.toml`（若已有 `[mcp_servers."懒蚂蚁"]`，先删掉旧表再粘贴）
3. 保存后重启 Codex / 新开 `codex` 会话

macOS 示例：

```toml
[mcp_servers."懒蚂蚁"]
command = "/Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp"
args = []

[mcp_servers."懒蚂蚁".env]
LAZYANT_DATA_DIR = "/Users/you/Library/Application Support/懒蚂蚁"
```

Windows：`command` 指向安装目录 `resources\lazyant-mcp.cmd`。  
Linux：`resources/lazyant-mcp`。

### CLI 快捷注册

```bash
codex mcp add "懒蚂蚁" -- /Applications/懒蚂蚁.app/Contents/Resources/lazyant-mcp
```

注册后检查 `~/.codex/config.toml`，确认存在 `[mcp_servers."懒蚂蚁".env]` 且 `LAZYANT_DATA_DIR` 与懒蚂蚁数据目录一致；缺失则手动补上。

### 开发态（源码仓库）

```toml
[mcp_servers."懒蚂蚁"]
command = "pnpm"
args = ["--dir", "/path/to/lazy-ant/apps/desktop", "cli:mcp"]

[mcp_servers."懒蚂蚁".env]
LAZYANT_DATA_DIR = "/Users/you/Library/Application Support/懒蚂蚁"
```

也可：

```bash
codex mcp add "懒蚂蚁" -- pnpm --dir /path/to/lazy-ant/apps/desktop cli:mcp
```

并在 `config.toml` 中补上 `LAZYANT_DATA_DIR` 环境变量。

### 验证

在 Codex 中发起对话，要求列出懒蚂蚁工具或调用 `get_article_draft`；工具列表应出现 `lazyant` 下的 22 个工具，以及 4 个 Prompt、4 个 Resource。若连接失败，检查启动器路径、`LAZYANT_DATA_DIR` 与懒蚂蚁是否曾成功启动过（确保 `userData` 目录存在）。

服务通过 **stdio** 与客户端通信；不要向 stdout 打印日志（会干扰协议）。

## 写前先读规范（推荐工作流）

外部 Agent（Cursor / Codex 等）在写懒蚂蚁产物前，应先调用对应只读规范工具，或**优先使用 MCP Prompt**（规范会在调大模型之前注入）。规范与技能库「复制提示词」同源（`market/skills/_builtin/*`），MCP 初始化时也会注入 server instructions。

### 推荐：MCP Prompt（先注入规范，再调大模型）

| Prompt       | 参数       | 落稿工具                                                            |
| ------------ | ---------- | ------------------------------------------------------------------- |
| `写文章`     | `写作要求` | `save_article_draft`                                                |
| `写图文`     | `写作要求` | `save_image_text_draft`                                             |
| `做文章主题` | `写作要求` | 新建 `save_personal_article_theme`；修改按 source 选择对应 `update` |
| `做图文主题` | `写作要求` | 新建 `save_personal_image_theme`；修改按 source 选择对应 `update`   |

在 Cursor 对话里选择上述 MCP Prompt 并填写 **写作要求**；客户端会先 `prompts/get` 把 SKILL 全文注入首条消息，再进入大模型生成。正文只写入 `save_*` 工具参数。

#### 写作要求：自然输入也算

用户不一定单独填 Prompt 参数表。常见写法是把需求写在同一条消息里，例如：

```text
/写文章 帮我写一篇懒蚂蚁功能介绍
/写文章 优化下面这篇文章（正文附后）
```

**Agent 应把 `/写文章`（或 `/写图文` 等）之后、同条消息中的说明与正文，整体视为「写作要求」。**

Cursor 有时只触发 Prompt、不把消息正文填入 `写作要求` 参数，会导致 `prompts/get` 报「写作要求 必填」。这**不代表用户没给要求**——Agent 应忽略该报错，用用户消息作为要求，改走 `get_*_spec` → 按规范生成 → `save_*` 落盘。

### Cursor：`@` 不会出现 Prompt；用 `/`

- **`@` 菜单**：用于文件、Rules、Docs 等上下文，**不会列出 MCP Prompt**（Cursor 产品设计如此）。
- **`/` 菜单**：可触发 MCP Prompt。若仍显示 `write_lazyant_article` 等英文名，说明 **lazyant MCP 子进程未刷新**（见下方「重连 MCP」）。

#### 重连 MCP（英文名变中文时必做）

1. **安装包用户**：更新懒蚂蚁到最新版后，Cursor → **Settings → MCP** → `lazyant` → **Restart**
2. **源码开发**：`cd apps/desktop && pnpm run build:mcp`，再 Restart MCP
3. 仍不对：**完全退出 Cursor 再打开**
4. 验证：MCP 说明里应出现「写文章」而非 `write_lazyant_article`

> 兼容：旧版英文 ID（如 `write_lazyant_article`）与参数名 `brief` 仍可用于 `prompts/get`，但列表应显示中文名。

### 备选：Resource @ 引用

| URI                                  | 说明              |
| ------------------------------------ | ----------------- |
| `lazyant://spec/article-markdown`    | 文章结构规范      |
| `lazyant://spec/image-text-markdown` | 图文分卡规范      |
| `lazyant://spec/article-theme`       | 文章主题 CSS 规范 |
| `lazyant://spec/image-theme`         | 图文主题 CSS 规范 |

### 备选：get___spec + save__

| 用户意图               | 先调用（读规范）               | 再调用（落盘）                                                                                  |
| ---------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------- |
| 写公众号/长文 Markdown | `get_article_markdown_spec`    | `save_article_draft`                                                                            |
| 写小红书等图文分卡     | `get_image_text_markdown_spec` | `save_image_text_draft`                                                                         |
| 做/改文章主题 CSS      | `get_article_theme_spec`       | `save_personal_article_theme` / `update_personal_article_theme` / `update_market_article_theme` |
| 做/改图文主题 CSS      | `get_image_theme_spec`         | `save_personal_image_theme` / `update_personal_image_theme` / `update_market_image_theme`       |

Agent 应先调用 `get_*_spec` 阅读规范，再按规范在 `save_*` 的 `content` / `themeCSS` 参数里生成产物，不要先在聊天气泡里写全文。

### 修改已有产物：必须原位更新

- 文章/图文：先 `get_*_draft` 读取当前定稿，基于原文修改，再调用 `save_*_draft`；固定覆盖同一个 `article-ready`。
- 文章主题：先 `list_article_themes` 确定 `source` + `folderId`，再 `get_article_theme` 读取完整主题；`personal` 调 `update_personal_article_theme`，`market` 调 `update_market_article_theme`。
- 图文主题：先 `list_image_themes` 确定 `source` + `folderId`，再 `get_image_theme` 读取完整主题；`personal` 调 `update_personal_image_theme`，`market` 调 `update_market_image_theme`。
- `folderId` 由 Agent 根据用户说的主题名或当前上下文自动获取，不向用户索要。只有多个同名候选无法消歧时，才请用户按主题名和来源选择。
- 只有用户明确要求“新建”或“再做一套”时才调用 `save_personal_*_theme`。个人主题同名 save 会更新原 `folderId`；市场主题同名会拒绝创建副本。

## 工具列表

### Prompts（推荐入口）

| Prompt       | 说明                                           |
| ------------ | ---------------------------------------------- |
| `写文章`     | 预置文章排版语法后写作（参数 `写作要求`）      |
| `写图文`     | 预置图文分卡规范后写作（参数 `写作要求`）      |
| `做文章主题` | 预置文章主题 CSS 规范后做皮（参数 `写作要求`） |
| `做图文主题` | 预置图文主题 CSS 规范后做皮（参数 `写作要求`） |

### Resources

| URI                                  | 说明              |
| ------------------------------------ | ----------------- |
| `lazyant://spec/article-markdown`    | 文章排版语法      |
| `lazyant://spec/image-text-markdown` | 图文卡片语法      |
| `lazyant://spec/article-theme`       | 文章主题 CSS 规范 |
| `lazyant://spec/image-theme`         | 图文主题 CSS 规范 |

### Tools

| 工具                            | 说明                                                    |
| ------------------------------- | ------------------------------------------------------- |
| `get_article_markdown_spec`     | 文章排版语法（cover / toc / PART / footer-cta）         |
| `get_image_text_markdown_spec`  | 图文卡片语法（`---` 分卡）                              |
| `get_article_theme_spec`        | 文章主题 CSS 规范（`#output`、样例 CSS）                |
| `get_image_theme_spec`          | 图文主题 CSS 规范（`.card-*`、样例 CSS）                |
| `get_article_draft`             | 读取文章定稿                                            |
| `save_article_draft`            | 新建/修改文章定稿（merge 语义）                         |
| `get_image_text_draft`          | 读取图文定稿                                            |
| `save_image_text_draft`         | 新建/修改图文定稿（`---` 分卡，至少 4 张卡，3 个 tags） |
| `list_workspace_skills`         | 列出 skill-packs                                        |
| `get_skill_pack`                | 读取 Skill 包                                           |
| `save_workspace_skill`          | 新建/更新 Skill（单文件或整包）                         |
| `remove_workspace_skill`        | 删除 Skill                                              |
| `list_article_themes`           | 市场 + 个人文章主题索引                                 |
| `get_article_theme`             | 读取文章主题包                                          |
| `save_personal_article_theme`   | 新建个人文章主题；市场同名时拒绝创建副本                |
| `update_personal_article_theme` | 按 `folderId` 修改                                      |
| `update_market_article_theme`   | 按 `folderId` 原位修改可写的市场主题                    |
| `list_image_themes`             | 市场 + 个人图文主题索引                                 |
| `get_image_theme`               | 读取图文主题包                                          |
| `save_personal_image_theme`     | 新建个人图文主题；市场同名时拒绝创建副本                |
| `update_personal_image_theme`   | 按 `folderId` 修改                                      |
| `update_market_image_theme`     | 按 `folderId` 原位修改可写的市场图文主题                |

## 数据路径

| 产物          | 路径（相对 userData）                   |
| ------------- | --------------------------------------- |
| 文章/图文定稿 | `workspace/articles/article-ready.json` |
| 个人文章主题  | `workspace/themes/article/<folderId>/`  |
| 个人图文主题  | `workspace/themes/image/<folderId>/`    |
| Skill         | `skill-packs/<skillId>/`                |

懒蚂蚁桌面端启动时会扫描 `workspace/themes/` 同步到「我的主题」；MCP 写入后若 App 已运行会自动刷新。

## 客户端接入

- **外部客户端**：通过本 MCP Server 访问 `product-services` 落盘层。
- 市场官方主题 / Skill **只读**；写回 market 目录仍为 dev-only，不暴露给 MCP。

## 示例

保存文章：

```json
{
  "title": "示例标题",
  "digest": "一句话摘要",
  "content": "# 正文\n\n完整 Markdown…",
  "tags": ["标签1", "标签2"]
}
```

保存图文（`content` 用 `---` 分卡）：

```json
{
  "title": "笔记标题",
  "digest": "八十到一百字摘要",
  "content": "钩子\n\n---\n\n经历\n\n---\n\n方法\n\n---\n\n提问",
  "tags": ["效率", "干货", "成长"]
}
```
