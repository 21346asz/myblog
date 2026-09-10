# 我的博客

Hugo + FixIt 主题的中文博客，部署在 GitHub Pages 上，push 即自动发布。

## 环境要求

- **Hugo extended**：已安装到 `D:\wArn\tool\hugo`，并加入用户 PATH（新终端直接可用 `hugo` 命令）
- **Dart Sass**：FixIt 主题用它编译样式，已安装到 `D:\wArn\tool\dart-sass`，并加入用户 PATH
- **Git**

> 提示：Hugo 0.166 不再内置 Sass 编译器，所以必须有 Dart Sass；两者都已配好，正常不用管。

## 日常写作三步

```bash
# 1. 新建文章（在 D:\MyBlog 下执行）
hugo new content post/我的新文章/index.md
# 然后编辑文件，把 draft: true 改为 false，写正文，图片放同目录

# 2. 本地预览
hugo server -D        # 打开 http://localhost:1313

# 3. 发布
git add -A
git commit -m "新增文章"
git push
```

push 后 GitHub Actions 自动构建发布，约一分钟后线上可见。

## 配置在哪改（FixIt 用 config/_default/ 目录管理配置）

| 想改什么 | 改哪里 |
| -------- | ------ |
| 博客标题 | `config/_default/hugo.toml` → `title` |
| 站点描述 | `config/_default/params.toml` → `description` |
| 首页作者卡片（名字、签名、头像、社交链接） | `config/_default/params.toml` → `[author]` / `[home.profile]` / `[social]` |
| 顶部导航菜单 | `config/_default/menus.toml` |
| 外观与功能（明暗主题、代码块、目录、搜索等） | `config/_default/params.toml` |
| 每页文章数、链接格式 | `config/_default/hugo.toml` / `config/_default/permalinks.toml` |
| 头像 / logo | 图片放 `assets/img/`，在 `params.toml` 的 `[author]` 里填 `avatar = "img/avatar.png"` |

## 文章写法

每篇文章是一个独立文件夹（Page Bundle），图片直接放在文章旁边：

```text
content/post/我的新文章/
├── index.md
├── cover.jpg       ← 封面图（在 front matter 里写 featured_image: "cover.jpg"）
└── screenshot.png  ← 正文配图
```

front matter 示例（`hugo new` 会自动生成一部分）：

```yaml
---
title: "我的新文章"
date: 2026-09-10T16:30:00+08:00
draft: false              # 改成 false 才会发布
description: "一句话摘要，会显示在列表和搜索里"
tags: ["标签A", "标签B"]
categories: ["分类"]
featured_image: "cover.jpg"   # 可选封面
---
```

## 首次部署到 GitHub（只需做一次）

1. 在 GitHub 上新建仓库（例如 `blog`；如果叫 `<用户名>.github.io` 就是主页仓库，本配置两种都支持，**不要**在 GitHub 上勾选初始化 README）。
2. 关联并推送：

   ```bash
   cd D:\MyBlog
   git remote add origin https://github.com/<你的用户名>/<仓库名>.git
   git push -u origin main
   ```

3. 打开仓库页面 → **Settings** → **Pages** → **Build and deployment** → Source 选择 **GitHub Actions**。
4. 等待 Actions 跑完（仓库的 Actions 标签页可以看到进度），博客地址：
   - 普通仓库：`https://<用户名>.github.io/<仓库名>/`
   - 主页仓库：`https://<用户名>.github.io/`

> 部署流程会自动安装 Dart Sass，无需额外设置。

## 常用维护命令

```bash
hugo --minify                                     # 本地完整构建，输出到 public/
hugo server -D                                    # 本地实时预览（含草稿）
git submodule update --remote themes/FixIt        # 升级 FixIt 主题（之后必须本地构建验证）
hugo env                                          # 查看当前 Hugo 版本
```

> 说明：构建时可能出现两条 WARN——一条是主题自带的 `imaging.quality` 弃用提示，
> 一条是主题 SCSS 里旧的斜杠除法提示。它们**都不影响使用**，属正常现象，
> 主题后续版本会修复。
