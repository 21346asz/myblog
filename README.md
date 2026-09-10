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
hugo new content posts/我的新文章/index.md
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

## 内容目录结构（怎么归档）

所有内容都在 `content/posts/` 下，按类型分层；**网页地址统一是 `/p/<文件夹名>/`**（与目录层级无关）：

```text
content/posts/
├── hello-world/                # 日常文章（随笔 / 技术 / 教程）——直接放这里
├── blog-guide/
└── writeups/                   # 靶机写up
    ├── hackmyvm/
    │   ├── easy/<靶机名>/
    │   ├── medium/<靶机名>/
    │   └── <靶机名>/           # 未标注难度的
    └── mazesec/
        ├── easy/<靶机名>/
        └── medium/<靶机名>/
```

| 要发什么 | 放哪里 | 新建命令 |
| -------- | ------ | -------- |
| 日常文章 | `content/posts/<文章名>/` | `hugo new content posts/我的新文章/index.md` |
| 靶机写up | `content/posts/writeups/<平台>/<难度>/<靶机名>/` | `hugo new content posts/writeups/hackmyvm/easy/靶机名/index.md` |

靶机写up 的 front matter 记得带上分类和标签（来源 + 难度）：

```yaml
categories: ["靶机复盘"]
tags: ["HackMyVM", "Easy"]
```

### ⚠️ 放错位置会出问题（必读）

**铁律：文章文件夹必须放在 `content/posts/` 下面（里面套几层都行）。**

放到 `content/` 根下的**其他**文件夹（比如 `content/hackthebox/`）**不会报错**，但会变成"独立栏目"，副作用是：网址变成 `/hackthebox/start/`、文章页**缺少日期/分类/标签/目录**、也**不上首页和归档**。

| 写法 | 网址 | 完整文章页 | 上首页 |
| ---- | ---- | ---------- | ------ |
| `content/posts/hackthebox/start/index.md` ✅ | `/p/start/` | 有 | 有 |
| `content/hackthebox/start/index.md` ❌ | `/hackthebox/start/` | 缺元信息区 | 无 |

另外 4 个常见坑（都不报错，但文章会"消失"或异常）：

1. **`draft: true`** —— 不会发布，改成 `false`。
2. **日期写成"未来"** —— 本站 `buildFuture=false`，未来日期的文章会被**静默跳过**（看起来像没发）。用 `hugo new` 生成的日期是当前时间，不会踩这个坑。
3. **文件夹名重复** —— 两篇会抢同一个 `/p/<文件夹名>/` 网址。改文件夹名，或在 front matter 里加 `slug: 新的名字`。
4. **front matter 语法错**（引号不配、冒号后没空格）—— 本地 `hugo` 会报错；**push 前先本地 `hugo --minify` 构建一次**最稳。

> 小提示：主题会自动把**标题首字母大写**（`Start（放在 posts 下）` 会显示成 `Start（放在 Posts 下）`），属正常现象，不用管。

## 文章写法

每篇文章是一个独立文件夹（Page Bundle），图片直接放在文章旁边：

```text
content/posts/我的新文章/
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

3. Pages **通常无需手动设置**：推送后 GitHub Actions 会自动启用 Pages（已配置 `enablement: true`）。
   如果第一次没成功，也可以手动来一次：仓库 **Settings → Pages**（直接访问
   `https://github.com/<用户名>/<仓库名>/settings/pages`），把 **Source** 设为 **GitHub Actions**。
4. 等待 Actions 跑完（仓库的 Actions 标签页可以看到进度），博客地址：
   - 普通仓库：`https://<用户名>.github.io/<仓库名>/`
   - 主页仓库：`https://<用户名>.github.io/`

> 注意：**只有公开仓库（Public）才能免费使用 Pages**。若仓库是 Private，Pages 设置里不会出现 Source 选项，
> 需要把仓库改为 Public（Settings → General → Danger Zone → Change visibility）或升级付费账号。
> 部署流程会自动安装 Dart Sass，无需额外设置。

## 常用维护命令

```bash
hugo --minify                                     # 本地完整构建，输出到 public/
hugo server -D                                    # 本地实时预览（含草稿）
git submodule update --remote themes/FixIt        # 升级 FixIt 主题（之后必须本地构建验证）
hugo env                                          # 查看当前 Hugo 版本
```

> 说明：构建现在是**零警告**的。此前主题早期写法引起的两条提示（`imaging.quality` 弃用、
> SCSS 斜杠除法）已在本项目中修复，不需要再关注。
