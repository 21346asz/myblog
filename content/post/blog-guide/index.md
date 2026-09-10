---
title: "博客使用指南：如何写作与发布"
date: 2026-09-10T16:20:00+08:00
draft: false
description: "写给未来的自己：新增文章、插入图片、本地预览、发布的完整流程。"
tags:
  - 教程
categories:
  - 指南
---

这篇文章就是本站的"说明书"，忘了流程随时回来看。

## 1. 新建一篇文章

在 `D:\MyBlog` 目录下执行：

```bash
hugo new content post/我的新文章/index.md
```

生成的文件开头有一段 front matter（两行 `---` 之间的部分）：

```yaml
---
title: "我的新文章"   # 标题
date: 2026-09-10T16:30:00+08:00
draft: true            # 改成 false 才会发布
description: "一句话摘要，会显示在列表和搜索里"
tags: ["标签A", "标签B"]
categories: ["分类"]
image: "cover.jpg"     # 可选：封面图（放在同目录下）
---
```

## 2. 插入图片

每篇文章是一个独立的文件夹（Page Bundle），图片直接放在文章旁边即可：

```text
content/post/我的新文章/
├── index.md
├── cover.jpg      ← 封面图
└── screenshot.png ← 正文配图
```

正文中引用：`![截图说明](screenshot.png)`

## 3. 本地预览

```bash
hugo server -D
```

打开 [http://localhost:1313](http://localhost:1313) 实时预览，保存即刷新。`-D` 表示连草稿（draft: true）一起显示。

## 4. 发布

```bash
git add -A
git commit -m "新增文章：我的新文章"
git push
```

push 之后 GitHub Actions 会自动构建并发布到 GitHub Pages，大约一分钟后生效。

## 5. 常用管理操作

- **改博客标题/副标题**：编辑 `hugo.yaml` 顶部的 `title` 和 `sidebar.subtitle`
- **改侧边栏头像**：把图片放到 `assets/img/avatar.png`，把 `hugo.yaml` 中 `sidebar.avatar.enabled` 改为 `true`
- **升级主题**：`git submodule update --remote themes/stack`
- **本地完整构建测试**：`hugo --minify`，结果输出到 `public/` 目录
