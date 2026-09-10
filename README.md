# 我的博客

Hugo + Blowfish 主题的中文博客，部署在 GitHub Pages 上，push 即自动发布。

## 环境要求

- Hugo extended（已安装到 `D:\wArn\tool\hugo`，并已加入用户 PATH；新开的终端直接可用 `hugo` 命令）
- Git

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

## 配置在哪改（Blowfish 用 config/_default/ 目录管理配置）

| 想改什么 | 改哪里 |
| -------- | ------ |
| 博客标题 / 站点描述 | `config/_default/languages.zh-cn.toml` → `title` / `description` |
| 首页作者卡片（名字、签名、社交链接） | `config/_default/languages.zh-cn.toml` → `[params.author]` |
| 顶部导航菜单 | `config/_default/menus.zh-cn.toml` |
| 外观与功能（配色、布局、代码复制、目录等） | `config/_default/params.toml` |
| 站点基础（每页文章数、链接格式） | `config/_default/hugo.toml` |
| 头像 / logo | 图片放 `assets/img/`，在 `languages.zh-cn.toml` 对应位置填路径 |

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

## 常用维护命令

```bash
hugo --minify                                    # 本地完整构建，输出到 public/
git submodule update --remote themes/blowfish    # 升级 Blowfish 主题（之后必须本地构建验证）
hugo env                                         # 查看当前 Hugo 版本
```

> 说明：本地 Hugo 为 0.166.0 extended，Blowfish（v3.6.0）官方声明支持到 0.165.0，
> 构建时会有一条版本范围 WARN——不影响使用，属正常现象。
