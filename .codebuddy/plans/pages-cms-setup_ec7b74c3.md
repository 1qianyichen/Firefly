---
name: pages-cms-setup
overview: 为 Firefly Astro 博客配置 Pages CMS，创建 .pages.yml 文件覆盖 posts/dynamic/spec 三个内容集合，并指导 GitHub App 连接。
todos:
  - id: create-pages-yml
    content: 创建项目根目录下的 .pages.yml 配置文件，定义 posts、dynamic、spec 三个内容集合的完整字段映射和媒体上传配置
    status: completed
  - id: create-guide-doc
    content: 创建 Pages CMS 部署指引文档，包含 GitHub App 注册步骤、Pages CMS 连接流程、以及 Cloudflare Pages / Vercel 部署建议
    status: completed
    dependencies:
      - create-pages-yml
---

## 用户需求

为 Firefly Astro 博客接入 Pages CMS（开源 Git-based 无头 CMS），通过可视化界面管理博客内容，无需每次手动编辑 Markdown 文件。

## 核心功能

- **Posts 集合管理**：创建/编辑/删除博客文章，支持 title、published、tags、category、draft、password 等全部字段，支持富文本编辑正文
- **Dynamic 集合管理**：管理"说说/动态"短文内容，支持 published、pinned、location 字段
- **Spec 页面管理**：管理"关于"、"友链"、"留言板"三个独立页面
- **媒体文件上传**：支持图片上传到 `src/content/posts/images/` 目录
- **GitHub 集成**：通过 Pages CMS 的 GitHub App 实现登录认证和内容自动提交

## 技术方案

### 实现策略

在项目根目录创建 `.pages.yml` 配置文件，定义三个内容集合（posts、dynamic、spec）和媒体上传路径。无需安装任何 npm 依赖——Pages CMS 是一个独立的 SaaS 平台，通过 GitHub App 直接读写仓库中的文件。

### .pages.yml 配置设计

#### 1. Posts 集合配置

- **type**: `collection`
- **path**: `src/content/posts`
- **format**: `yaml-frontmatter`
- **filename**: `{primary}.md`（通过自定义 slug 字段命名文件）
- **subfolders**: `true`（支持 guide/ 等子目录）
- **fields**: 映射 19 个面向用户的字段，排除 prevTitle/prevSlug/nextTitle/nextSlug 四个内部字段
- `title`: string（必填）
- `slug`: string（文件名标识，必填）
- `published`: date（必填，日期格式 YYYY-MM-DD）
- `updated`: date（可选）
- `draft`: boolean（默认 false）
- `description`: text（多行文本）
- `image`: string（支持网络URL和相对路径，提示支持 "api" 特殊值）
- `tags`: list of string
- `category`: string
- `lang`: string
- `pinned`: boolean
- `author`: string
- `sourceLink`: string
- `licenseName`: string
- `licenseUrl`: string
- `comment`: boolean（默认 true）
- `password`: string（用于文章加密）
- `passwordHint`: string
- `body`: rich-text（Markdown 正文编辑）

#### 2. Dynamic 集合配置

- **type**: `collection`
- **path**: `src/content/dynamic`
- **format**: `yaml-frontmatter`
- **filename**: `{published}-.md`（时间戳命名模式）
- **fields**: published(date)、pinned(boolean)、location(string)、body(rich-text)

#### 3. Spec 页面配置

- **type**: `group` 包裹三个 `file` 类型
- **files**: about.md、friends.mdx、guestbook.md
- **format**: `yaml-frontmatter`
- **fields**: title(string)、description(string)、body(rich-text)

#### 4. Media 配置

- **input**: `src/content/posts/images`（图片上传至 posts 图片目录）
- **output**: `./images`（内容中写入相对路径）
- **extensions**: [png, jpg, jpeg, webp, gif, avif, svg]
- **rename**: `safe`（安全的文件名处理）

### 关键技术决策

- **slug 字段处理**：Pages CMS 默认用 `{primary}.md` 命名文件，将 slug 作为 primary 字段，方便通过 slug 查找和管理文章
- **日期格式**：Pages CMS 的 date 字段默认输出 `YYYY-MM-DD` 格式，与项目 frontmatter 中的日期格式一致
- **Rich-text 正文**：使用 Pages CMS 内置的 rich-text 编辑器，自动将 Markdown 正文存储到 `body` 字段
- **Spec 页面用 file 类型**：因为 about/friends/guestbook 是固定页面而非集合，用 file 类型可避免误创建新文件

## 使用的 Agent 扩展

### SubAgent

- **code-explorer**
- 目的：在生成 .pages.yml 前确认所有三个内容集合的精确目录结构和示例文件的 frontmatter 格式
- 预期结果：获取 posts、dynamic、spec 目录下文件的完整 frontmatter 字段，确保配置与项目 100% 匹配