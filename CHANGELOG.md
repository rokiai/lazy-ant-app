# 更新日志

本文件遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 格式。桌面端 LazyAnt 默认读取项目根目录的此文件。

## [Unreleased]

### 新增

- 编辑器 **文章 / 图文** 切换：图文模式自研卡片预览（主题、横线 `---` 拆分、尺寸）
- 图文 **导出 PNG 到本地** `output/xhs-cards/<时间戳>/`（含 `manifest.json` 标题与话题）
- 侧栏 **更新日志**：渲染本地 `CHANGELOG.md`
- 侧栏 **设置**：默认 Skill、各平台主题色、手动/自动工作流
- 侧栏 **历史发布**：记录标题、平台、CLI 日志与成败
- 同步平台展示各平台官方 Logo
- 工作台四步向导（CDP → 生成 → 预览定稿 → 同步发布）
- Skill 模板树形查看与编辑

### 改进

- 发布前 CDP 与各平台登录态检测
- Chrome 调试启动脚本后台运行并轮询 CDP 就绪

## [1.0.0] - 2026-07-20

### 新增

- Electron 桌面端 LazyAnt：Skill 驱动写作 + Markdown 编辑器
- CLI：`run` / `publish` 与掘金、公众号、知乎适配器
- 内置 Skill：`hot-news-writer`
