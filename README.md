<div align="center">

<img src="./assets/icon.png" alt="LazyAnt" width="128" />

# 懒蚂蚁 · LazyAnt

**把内容整理好，然后发出去**

Skill 驱动写作 · 文章与图文双模式 · 一键多平台发布

macOS · Windows · Linux

<br />

[**下载安装包**](https://github.com/rokiai/lazy-ant-app/releases/latest) · [**更新日志**](./CHANGELOG.md)

</div>

## 下载

前往 **[GitHub Releases](https://github.com/rokiai/lazy-ant-app/releases/latest)** 获取最新版，或点击下方链接直接下载：

| 系统        | 架构                    | 安装包              | 下载                                                                                          |
| :---------- | :---------------------- | :------------------ | :-------------------------------------------------------------------------------------------- |
| **macOS**   | Apple Silicon（M 系列） | `LazyAnt-arm64.dmg` | [**下载**](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-arm64.dmg) |
| **macOS**   | Intel                   | `LazyAnt-x64.dmg`   | [**下载**](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-x64.dmg)   |
| **Windows** | 64 位                   | `LazyAnt-setup.exe` | [**下载**](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-setup.exe) |
| **Linux**   | 64 位                   | `LazyAnt.AppImage`  | [**下载**](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt.AppImage)  |
| **Linux**   | Debian / Ubuntu         | `lazyant_amd64.deb` | [**下载**](https://github.com/rokiai/lazy-ant-app/releases/latest/download/lazyant_amd64.deb) |

> 链接始终指向 **最新 Release**。若尚未发版，请先到 [Releases 页](https://github.com/rokiai/lazy-ant-app/releases) 查看；发版后上表链接即可直接下载。
>
> 当前安装包**尚未做 Apple 开发者签名 / 公证**，从网上下载后系统可能提示「无法验证开发者」，见下方安装说明。

## 安装说明

### macOS

1. 按芯片下载对应 DMG：[Apple Silicon](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-arm64.dmg) · [Intel](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-x64.dmg)
2. 打开 DMG，把 **LazyAnt** 拖进「应用程序」
3. 从「应用程序」启动

若提示「已损坏，无法打开」，在终端执行：

```bash
xattr -cr /Applications/LazyAnt.app
open /Applications/LazyAnt.app
```

若提示「无法验证开发者」：右键 App →「打开」→ 再点「打开」；或在「系统设置 → 隐私与安全性」中点「仍要打开」。

### Windows

1. 运行 [LazyAnt-setup.exe](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt-setup.exe)
2. 若出现 SmartScreen：点「更多信息」→「仍要运行」

### Linux

- **AppImage**：`chmod +x LazyAnt.AppImage && ./LazyAnt.AppImage`（[下载](https://github.com/rokiai/lazy-ant-app/releases/latest/download/LazyAnt.AppImage)）
- **deb**：`sudo dpkg -i lazyant_amd64.deb`（[下载](https://github.com/rokiai/lazy-ant-app/releases/latest/download/lazyant_amd64.deb)，缺依赖时再 `sudo apt -f install`）

---

## 这是什么

**懒蚂蚁（LazyAnt）** 是面向内容创作者的桌面应用。从选题灵感、AI 写稿、排版预览，到同步各平台草稿箱——一条流水线在一个窗口里完成。

不用在浏览器标签页、笔记软件和各个创作者后台之间来回切换。写好、排好、选好账号，点发布即可。

<br />

<div align="center">
  <img src="./assets/控制台.png" alt="懒蚂蚁控制台" width="920" />
  <br />
  <sub>首页控制台：发布准备、助手协作、快捷入口一览</sub>
</div>

<br />

## 功能一览

|                |                                                     |
| :------------- | :-------------------------------------------------- |
| **文章编辑**   | Markdown 长文稿，封面 / 目录 / 结构化排版，实时预览 |
| **图文编辑**   | Markdown 拆卡，机型预览，导出 PNG 或直接发布        |
| **主题库**     | 文章主题 + 图文主题，市场浏览、收藏、一键套用       |
| **Skill 模板** | 提示词市场与我的模板，写作方法可复用                |
| **自动化**     | 选 Skill → AI 生成 → 多平台发布，配置一次反复跑     |
| **选题灵感**   | 多平台热榜聚合，快速找到值得写的题                  |
| **账号管理**   | 发布平台与写作模型，集中授权与登录态检测            |

## 文章编辑

Markdown 长文稿写作，左侧编辑、右侧实时预览。支持封面块、目录、高亮等结构化语法；可按小红书创作者后台机型预览排版效果，定稿后一键发布到已选账号。

<table>
  <tr>
    <td width="50%" align="center">
      <img src="./assets/文章编辑区1.png" alt="文章编辑 - 分栏预览" width="100%" />
      <br />
      <sub>分栏编辑 + 实时预览</sub>
    </td>
    <td width="50%" align="center">
      <img src="./assets/文章编辑区2.png" alt="文章编辑 - 发布面板" width="100%" />
      <br />
      <sub>机型预览 + 多账号发布</sub>
    </td>
  </tr>
</table>

## 图文编辑

把 Markdown 做成多张图文卡片。长文自动拆分，或用 `---` 横线手动分卡；支持剪贴板复制、PNG / SVG 导出，以及按平台机型预览后同步发布。

<div align="center">
  <img src="./assets/图文编辑区.png" alt="图文编辑" width="920" />
  <br />
  <sub>图文编辑：Markdown 写卡、机型预览、导出与发布</sub>
</div>

## 文章主题库

内置多种文章排版主题，在主题市场浏览、收藏、设为默认；写文章时随时切换，同一篇稿子秒变不同风格。

<div align="center">
  <img src="./assets/文章主题模版.png" alt="文章主题市场" width="920" />
  <br />
  <sub>文章主题市场：经典黑白、霁青手札、摸鱼绿、橄榄手记、红白等</sub>
</div>

<br />

<table>
  <tr>
    <td width="33%" align="center">
      <img src="./assets/文章主题1.png" alt="文章主题 - 摸鱼票据风" width="100%" />
      <br />
      <sub>摸鱼票据风</sub>
    </td>
    <td width="33%" align="center">
      <img src="./assets/文章主题2.png" alt="文章主题 - 霁青手札" width="100%" />
      <br />
      <sub>霁青手札</sub>
    </td>
    <td width="33%" align="center">
      <img src="./assets/文章主题3.png" alt="文章主题 - 红白色系" width="100%" />
      <br />
      <sub>红白色系</sub>
    </td>
  </tr>
</table>

## 图文主题库

图文卡片同样有丰富的主题皮肤：手账、记事本、玻璃拟态、复古打字机、梦幻渐变等，换主题即换整套卡片视觉。

<div align="center">
  <img src="./assets/图文主题模版.png" alt="图文主题市场" width="920" />
  <br />
  <sub>图文主题市场：玻璃拟态、儿童童话、复古打字机、极简黑白等</sub>
</div>

<br />

<table>
  <tr>
    <td width="33%" align="center">
      <img src="./assets/图文主题1.png" alt="图文主题 - 活页手账" width="100%" />
      <br />
      <sub>活页手账</sub>
    </td>
    <td width="33%" align="center">
      <img src="./assets/图文主题2.png" alt="图文主题 - 方格记事本" width="100%" />
      <br />
      <sub>方格记事本</sub>
    </td>
    <td width="33%" align="center">
      <img src="./assets/图文主题3.png" alt="图文主题 - 梦幻渐变" width="100%" />
      <br />
      <sub>梦幻渐变</sub>
    </td>
  </tr>
</table>

## Skill 模板

提示词模板市场 + 我的模板：复制到外部 AI 生成内容，再贴回编辑器排版发布。支持按场景筛选（公众号、插画、去 AI 味等），收藏的 Skill 还可用于自动化任务。

<div align="center">
  <img src="./assets/skill模板.png" alt="Skill 模板" width="920" />
  <br />
  <sub>Skill 模板：市场浏览、收藏、复制提示词、导入自定义模板</sub>
</div>

## 自动化流水线

选 Skill → AI 生成 → 多平台发布。为小红书、微博、知乎、公众号等平台各建一条任务，配置一次，有新选题时一键重跑。

<div align="center">
  <img src="./assets/任务介绍.png" alt="自动化任务" width="920" />
  <br />
  <sub>自动化任务：按平台配置 Skill 流水线，一键运行</sub>
</div>

## 选题灵感

抖音、小红书、知乎、B 站、36 氪、V2EX 等平台热榜聚合展示，帮你快速找到值得写的选题。后续还将支持关注列表与博主动态订阅。

<div align="center">
  <img src="./assets/热点.png" alt="选题与热榜" width="920" />
  <br />
  <sub>选题与关注：多平台热榜一站浏览</sub>
</div>

## 账号与模型

在一个地方管理各平台发布账号和写作模型授权。只检测你添加的目标，不会偷偷扫描未登录的平台。

<div align="center">
  <img src="./assets/账号管理.png" alt="账号管理" width="920" />
  <br />
  <sub>账号与模型：平台登录态、模型授权集中管理</sub>
</div>

## 支持的平台

掘金 · 微信公众号 · 知乎 · 语雀 · 百家号 · 小红书 · 微博 等（持续增加中）

## 快速上手

1. 打开应用，在「账号」里添加目标平台并完成登录
2. 在「模型」里授权写作用的 AI（如豆包、Kimi）
3. 从首页进入「发文章」或「做图文」，选主题、写内容、预览定稿
4. 点「发布」同步到平台草稿箱，或在「自动」里配置 Skill 流水线
