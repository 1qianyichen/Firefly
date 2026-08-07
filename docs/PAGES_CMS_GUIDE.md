# Pages CMS 部署指引

## 什么是 Pages CMS？

[Pages CMS](https://pagescms.org/) 是一个开源的无头 CMS，直接操作你 GitHub 仓库中的 Markdown 文件。它不需要数据库，所有内容变更都会通过 Git 提交到你的仓库中。

## 第一步：创建 GitHub App

Pages CMS 通过 GitHub App 来访问你的仓库。以下是创建步骤：

### 1.1 登录 Pages CMS

1. 打开 [Pages CMS 官网](https://pagescms.org/)
2. 点击右上角 **"Sign in"**，使用 GitHub 账号授权登录

### 1.2 配置 GitHub App

Pages CMS 提供了两种方式创建 GitHub App：

**方式一：使用内置助手（推荐）**

在 Pages CMS 的管理界面中，导航到设置页面，使用内置的 GitHub App 创建助手，它会自动完成大部分配置。

**方式二：手动创建**

1. 打开 [GitHub Apps 设置](https://github.com/settings/apps)
2. 点击 **"New GitHub App"**
3. 填写应用信息：
   - **GitHub App name**: `Pages CMS - Firefly Blog`
   - **Homepage URL**: 你的博客地址（如 `https://blog.cuteleaf.cn`）
   - **Callback URL**: `https://pagescms.org/api/auth/callback`
   - **Webhook URL**: `https://pagescms.org/api/webhook`
   - **Webhook secret**: 随机生成一个密钥并保存

4. 配置仓库权限（Repository permissions）：
   | 权限 | 级别 |
   |------|------|
   | Administration | Read and write |
   | Actions | Read and write |
   | Contents | Read and write |
   | Metadata | Read only |

5. 订阅事件（Subscribe to events）：
   - Repository
   - Push
   - Delete

6. 点击 **"Create GitHub App"**

7. 生成并下载私钥（Private key），保存备用

### 1.3 安装 GitHub App 到仓库

1. 在 GitHub App 设置页面，点击左侧 **"Install App"**
2. 选择你的博客仓库（`CuteLeaf/Firefly` 或你的 fork）
3. 点击 **"Install"**

## 第二步：在 Pages CMS 中连接仓库

1. 登录 [Pages CMS](https://pagescms.org/)
2. 点击 **"+ Add site"** 或类似按钮
3. 选择你的博客仓库
4. 选择分支（通常为 `master` 或 `main`）
5. Pages CMS 会自动读取仓库中的 `.pages.yml` 配置文件

## 第三步：验证配置

连接成功后，你应该能在 Pages CMS 后台看到三个内容集合：

- **博客文章** — 创建和管理所有博文
- **说说/动态** — 管理短文动态
- **特殊页面** — 管理"关于我"、"友情链接"、"留言板"

## 第四步：部署博客

你的博客需要部署上线才能让读者看到。项目已配置以下两个部署平台，任选其一：

### Cloudflare Pages（推荐，已有配置）

1. 确保代码已推送到 GitHub
2. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
3. 进入 **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
4. 选择你的 GitHub 仓库
5. 构建设置：
   - **Build command**: `pnpm build`
   - **Build output directory**: `dist`
6. 环境变量（可选）：
   - `NODE_VERSION`: `22`

### Vercel

1. 确保代码已推送到 GitHub
2. 登录 [Vercel](https://vercel.com/)
3. 点击 **"Add New..."** → **"Project"**
4. 选择你的 GitHub 仓库
5. Vercel 会自动识别 Astro 项目，使用默认配置即可

## 工作流程

```
你写文章 (Pages CMS 后台)
    ↓
保存 → 自动 Git commit 到 GitHub 仓库
    ↓
GitHub 触发 CI/CD (Cloudflare Pages / Vercel)
    ↓
自动构建并部署到线上
    ↓
读者看到最新内容
```

## 注意事项

1. **首次使用前**：确保 `.pages.yml` 已推送到 GitHub 仓库
2. **图片上传**：通过 Pages CMS 上传的图片会自动保存到 `src/content/posts/images/` 目录
3. **草稿文章**：将 `draft` 设为 `true` 的文章不会在博客上显示
4. **文章加密**：设置 `password` 字段后，读者需要输入密码才能查看
5. **本地开发**：在 Pages CMS 中编辑后，本地执行 `git pull` 即可同步内容

## 常见问题

**Q: Pages CMS 连接不上我的仓库？**
A: 检查 GitHub App 是否已正确安装到你的仓库，确保 Contents 权限为 Read and write。

**Q: 创建的文章没有出现在博客上？**
A: 检查 `draft` 是否被设为 `true`，以及 `published` 日期是否正确。

**Q: 图片上传后不显示？**
A: 确认 `output` 路径配置正确。上传到 `src/content/posts/images/` 的图片在文章中使用 `./images/filename.avif` 引用。
