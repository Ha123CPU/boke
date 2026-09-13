---
title: "用 Hugo 和 GitHub Pages 搭建静态博客全过程"
date: 2026-09-13T22:30:00+08:00
tags: [Hugo, GitHub Pages, 静态博客, 教程]
author: "Ha123CPU"
---

## 背景

博客一直是技术人员记录思考的好地方。过去我可能用过 WordPress、Typecho 之类的动态博客，但维护数据库、更新 PHP 版本、防注入这些事儿其实挺麻烦的。最近决定换到静态博客方案：**Hugo 生成静态 HTML，GitHub Pages 托管**，完全免费、不用服务器、部署也简单。

这个post本身就是在这个博客上写的第一篇文章，顺便把搭建过程记录下来，供以后参考，也给有类似需求的人一个可操作的参考。

## 为什么选 Hugo + GitHub Pages

Hugo 的优势很明显：

- **快**：Hugo 是用 Go 写的，编译速度极快。哪怕几百篇文章，生成也就是几十毫秒级别。
- **部署简单**：生成出来的纯 HTML 可以放在任何静态托管上。GitHub Pages 是免费的，托管在 GitHub 的仓库里，通过 PR 发布文章也符合 Git Workflow。
- **主题丰富**：Hugo 主题生态很活跃，挑一个颜值和功能都能接受的主题基本不费功夫。我选了 `hugo-theme-even`，清爽、阅读体验好，中文友好。
- **扩展支持（Extended）**：Hugo Extended 版支持 SCSS 编译，很多主题都依赖这个。装主题时一定要注意。

GitHub Pages 的优势：

- 免费、无需额外服务器
- 和 GitHub 仓库绑定，PR 机制天然适合博客发布流程
- 默认支持 HTTPS（现在强制了）

## 准备工作

### 1. GitHub 账号 + Personal Access Token

GitHub 上有一个账号后，为了让本地能读写仓库，通常需要一个 Personal Access Token（PAT）。使用场景：

- 创建仓库（REST API）
- push/pull 认证
- 设置 GitHub Pages 来源等

Token 创建时要根据需要选 scope。这次我用的是具有仓库读写权限的 token。拿到 token 后，本地可以通过 HTTPS URL 的方式嵌入 token进行操作，也可以配 `.netrc` 或在 `git credential` 里存储。

> 注意：token 是敏感凭据，别泄露。示例里展示的 token 是我自己的测试 token，不要拿去用。

### 2. 安装 Hugo Extended

Windows 下可以通过几个渠道安装 Hugo：

- `winget install Hugo.Hugo`（但这是非 Extended 版）
- 官方 GitHub Release 里下载 `hugo_extended_xxx_windows-amd64.zip`

主题如果用到了 SCSS，就需要 Hugo Extended。所以推荐直接用 Extended 版。

安装好后终端里运行：

```bash
hugo version
```

看到类似 `hugo v0.166.0-extended...` 就算装好了。

### 3. 创建一个新的 GitHub 仓库

博客用一个独立的仓库。比如我创建了 `boke`（就是“博客”的拼音缩写）。仓库名不重要，但为了好记我选了这个。

创建方式可以：

- 在 GitHub 网页端手动创建
- 用 GitHub REST API 通过脚本创建

这次我是通过 API 创建的，创建参数大致是：

- owner: 用户名
- repo: 仓库名
- private: false（博客一般public）
- description: 选填

创建后得到一个 HTTPS 地址，比如 `https://github.com/Ha123CPU/boke.git`。

### 4. 本地初始化 Hugo 站点

在本地某个目录下（例如 `D:/hermes_code/boke`）运行：

```bash
hugo new site boke
```

这一步会生成：

- `content/` ——Markdown 文章放在 `content/posts/` 下
- `themes/` ——Hugo 主题
- `hugo.toml` ——站点配置（Hugo 怕是最近把 config 文件名改到这个了，老教程可能是 `config.toml`，注意版本）
- `public/` ——构建输出目录

### 5. 选主题并安装

Hugo 主题可以：

- 直接下载 zip 到 `themes/`
- 用 Hugo 模块管理（go.mod 方式）
- 用 git submodule

我用的是 Hugo 模块方式，把 `hugo-theme-even` 作为模块引入：

```bash
cd boke
hugo mod init github.com/Ha123CPU/boke
hugo mod add github.com/onesandzeros/hugo-theme-even
```

注意锁定版本，避免日后主题更新导致布局变化。`go.mod` 里会记录依赖版本。

安装完后在 `hugo.toml` 里指定主题：

```toml
theme = 'even'
```

### 6. 配置 hugo.toml

配置项根据主题和需求而定。基本的包括：

- `baseURL`：站点最终地址，比如 `https://Ha123CPU.github.io/boke/`
- `languageCode`、`title`、`author`
- `paginate`：每页文章数
- 菜单配置（`menu.main`）
- 主题参数（`[params]`），有些主题有自己特定的参数
- 语法高亮、Markup 选项

特别注意：**baseURL 必须是真实部署地址**，否则生成的链接会指向本地或错误域名，导致 GitHub Pages 上资源加载失败。

### 7. 写第一篇文章

格式：Markdown，放在 `content/posts/` 下。

Frontmatter 示例（YAML 风格）：

```yaml
---
title: "文章标题"
date: 2026-09-13T22:30:00+08:00
tags: [标签1, 标签2]
author: "作者名"
---
```

正文是普通 Markdown。每篇文章一个文件。

### 8. 构建 Hugo 站点

```bash
hugo
```

这一步会读取 `content/`，用主题模板渲染，输出到 `public/`。

构建完成后，`public/` 里的文件就是最终要部署的静态站点。

### 9. 部署到 GitHub Pages（手动方式）

最简单的方式：把 `public/` 的内容推到仓库的某个目录（比如 `/docs`），然后在仓库设置里把 GitHub Pages 的 source 设为 `main` 分支的 `/docs` 文件夹。

操作步骤：

1. 把 `public/` 重命名或复制为 `docs/`
2. 创建 `.nojekyll` 文件在 `docs/` 根目录（告诉 GitHub Pages 不要用 Jekyll 处理）
3. 提交 `docs/` 到仓库
4. 在 GitHub 仓库 Settings → Pages 中设置 source 为 main /docs

或者直接用 GitHub Pages API 设置 source：

```bash
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"source": {"branch": "main", "path": "/docs"}}' \
  https://api.github.com/repos/USERNAME/REPO/pages
```

提交后 GitHub 会自动构建，成功后就能通过 `https://USERNAME.github.io/REPO/` 访问。

### 10. 后续发布流程

博客建立之后，每次发布新文章的典型流程：

1. 在 `content/posts/` 下创建新 Markdown 文件
2. 写 frontmatter（title, date, tags, author）和正文
3. 本地执行 `hugo`，构建更新的 `public/`
4. 更新 `docs/`（或改用 Actions 自动构建）
5. 提交到仓库的分支，开 PR
6. 合并到 `main` 后 GitHub Pages 自动部署

这样博客内容更新就完全基于 Git、通过 PR 审核后发布，适合个人博客或小团队博客。

## 遇到过的问题

- **Hugo 安装版本问题**：winget 的 Hugo 是非 Extended 版，装了不能编译 SCSS 主题。解决方法是手动下载 Extended 版。
- **主题目录的 gitlink 问题**：Hugo 初始化后主题目录自带 `.git`，直接 `git add` 会把主题变成嵌入式仓库引用（gitlink），导致仓库里主题成了指针而不是文件。解决方法是删除主题目录里的 `.git`，再 `git add` 作为普通目录。
- **baseURL 配置错误**：一开始配置成了 localhost，导致生成页面里的链接全是本地地址，GitHub Pages 上面资源加载失败。更正为真实部署地址后重建就好了。
- **.nojekyll 遗漏**：GitHub Pages 默认尝试用 Jekyll 构建站点，如果没有 `.nojekyll` 标记，可能会因为 Hugo 输出的某些文件结构而构建失败。根目录下加这个空文件是个好习惯。
- **草稿文章 (draft: true)**：Hugo 文章如果设置了 `draft: true`，`hugo` 命令不会发布到 public 里。初次测试文章忘改这个，导致重建后文章页面消失。

## 结语

用 Hugo + GitHub Pages 搭博客，整个过程比预想中顺利。关键点在于：

- Hugo Extended 版的安装
- 主题的正确引入方式
- baseURL 与部署地址一致
- `.nojekyll` 和草稿状态的处理

现在博客已经跑起来了，地址是：

**https://ha123cpu.github.io/boke/**

后面就可以专心写内容了。
