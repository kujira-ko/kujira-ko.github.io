+++
title = '从零搭建个人网站：Hugo + GitHub Pages + Cloudflare 通用攻略'
date = 2026-10-18T08:00:00+08:00
draft = false
description = '一篇面向所有人的通用建站攻略：Hugo 生成静态网站，GitHub Pages 免费托管，Cloudflare 接管 DNS 与 HTTPS，全程零成本。'
tags = ['Hugo', 'GitHub Pages', 'Cloudflare', '教程']
ShowToc = true
+++

想拥有一个自己的网站，但不想买服务器、不想备案、不想被域名托管费劝退？这篇攻略帮你用 **Hugo + GitHub Pages + Cloudflare** 零成本搭出一个体面的个人网站。

<!--more-->

## 这套方案是什么

```
你的电脑（写内容）
   │  git push
   ▼
GitHub 仓库 ──► GitHub Actions 自动构建 ──► GitHub Pages 托管
                                                  │
                                    Cloudflare DNS + CDN + HTTPS
                                                  │
                                          https://你的域名
```

- **Hugo**：静态网站生成器。你用 Markdown 写文章，它生成纯 HTML，速度快、不用数据库。
- **GitHub Pages**：GitHub 提供的免费静态托管，自带全球 CDN 和免费 HTTPS。
- **Cloudflare**：免费 DNS 托管 + CDN + 免费 HTTPS 证书，让你的域名"干干净净"地上线。

**为什么推荐这套**：免费；零运维（推代码即发布）；网站文件有版本管理；电脑关机网站也活着（因为它住在 GitHub 云端）。

## 准备工作

| 材料 | 说明 |
|---|---|
| GitHub 账号 | 免费注册 https://github.com |
| Git 客户端 | https://git-scm.com/downloads 安装 |
| 域名（可选） | 想用自定义域名再买，几十块一年；不买就先 `用户名.github.io` |
| 代码编辑器 | VSCode 或任意文本编辑器 |

## 第一步：安装 Hugo

安装 **Extended（扩展版）**，功能最全，一劳永逸。

```bash
# Windows（PowerShell）
winget install Hugo.Hugo.Extended

# macOS
brew install hugo

# Linux（Ubuntu/Debian）
sudo snap install hugo
```

安装后验证：

```bash
hugo version
```

## 第二步：创建站点与主题

```bash
# 新建站点（my-site 是你的项目文件夹名）
hugo new site my-site
cd my-site

# 安装主题（以 PaperMod 为例）
git clone https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

把主题写进配置（`hugo.toml`）：

```toml
baseURL = 'https://你的用户名.github.io/'   # 或你的域名
languageCode = 'zh-cn'
title = '我的网站'
theme = 'PaperMod'
```

## 第三步：写第一篇文章

```bash
hugo new posts/hello-world.md
```

编辑生成的 `content/posts/hello-world.md`，认识一下"头部信息"（Front Matter）：

```markdown
---
title: '你好，世界'
date: 2026-10-18
draft: false        # false = 正式发布；true = 草稿
---

正文用 Markdown 写！
```

**关键规则**：`title` 要加引号；`draft` 是布尔值（`true`/`false`），**不能加引号**；日期不能写未来时间，否则不发布。

## 第四步：本地预览

```bash
hugo server -D
```

浏览器打开 http://localhost:1313 就能实时预览效果（改文件自动刷新）。`-D` 表示连草稿一起显示。预览满意再发布，别把半成品扔上线。

## 第五步：推到 GitHub

1. 在 GitHub 新建公开仓库，名字随意（如 `my-site`）
2. 本地关联并推送：

```bash
git init
git branch -M main
git add .
git commit -m "init: first commit"
git remote add origin https://github.com/你的用户名/my-site.git
git push -u origin main
```

## 第六步：GitHub Actions 自动发布

在项目里新建 `.github/workflows/hugo.yml`，内容如下（直接抄）：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        run: hugo --minify
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

然后**在 GitHub 网页上**：仓库 → Settings → Pages → Build and deployment → Source 选择 **GitHub Actions**。

推送代码（或手动触发 Actions），大约 2~3 分钟后访问 `https://你的用户名.github.io/` —— 网站上线！

## 第七步（可选）：自定义域名 + Cloudflare

1. **注册域名**（阿里云、腾讯云、Cloudflare 均可，`.com` 常见且便宜）
2. **把域名加入 Cloudflare**（Add a site → Free 计划），记下它给你的两个 Nameserver
3. **去域名注册商把 DNS 服务器改成 Cloudflare 给的 Nameserver**（生效几十分钟到 24 小时）
4. Cloudflare 后台 → DNS → Records，添加：
   - 类型 `CNAME`，名称 `@`，目标 `你的用户名.github.io`，开橙色云朵（Proxied）
   - 可选再加一条 `www` 的同款 CNAME
5. Cloudflare → SSL/TLS → 加密模式选 **Full**
6. GitHub 仓库 → Settings → Pages → Custom domain 填你的域名 → Save，再勾选 Enforce HTTPS

等证书签发（几分钟到几小时），访问 `https://你的域名` —— 大功告成！

## 日常更新

```bash
hugo new posts/新文章.md        # 写
hugo server -D                  # 看（可选）
git add . && git commit -m "add: 新文章" && git push   # 发
```

推完等 Actions 构建，刷新域名就能看到新内容。**写 → 看 → 推**，就这一个习惯。

## 常见问题

| 症状 | 排查 |
|---|---|
| 构建失败 | Actions 页面看日志；检查 workflow 缩进是否用了 Tab（要用空格） |
| 页面没有样式 | 主题没生效：确认 `hugo.toml` 的 `theme = 'xxx'` 与 `themes/` 文件夹名完全一致（区分大小写） |
| 新文章不显示 | 检查 `draft` 是否为 `false`；预览时加 `-D` |
| 自定义域名 404 | DNS 没生效或 CNAME 没配对；等待传播 |
| HTTPS 报错 | Cloudflare SSL 模式选 Full；GitHub 勾选 Enforce HTTPS |

## 费用清单

| 项目 | 费用 |
|---|---|
| Hugo / Git / GitHub Pages / Actions / Cloudflare | 全部免费 |
| 域名 | 约 ¥50~90 / 年 |
| 服务器、备案 | 都不需要 |

祝建站顺利！🎉