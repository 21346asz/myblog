# 我的博客

Hugo + Stack 主题的中文博客，部署在 GitHub Pages 上，push 即自动发布。

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

## 站点设置在哪改

| 想改什么 | 改哪里 |
| -------- | ------ |
| 博客标题 | `hugo.yaml` → `title` |
| 侧边栏副标题 | `hugo.yaml` → `params.sidebar.subtitle` |
| 头像 | 图片放 `assets/img/avatar.png`，`hugo.yaml` → `params.sidebar.avatar` 填 `img/avatar.png` |
| 侧边栏 GitHub 链接 | `hugo.yaml` → `menu.social` 里的 URL |
| 每页文章数 | `hugo.yaml` → `pagination.pagerSize` |
| 侧边栏菜单 | `hugo.yaml` → `menu.main` |

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
hugo --minify                                  # 本地完整构建，输出到 public/
git submodule update --remote themes/stack     # 升级 Stack 主题
hugo env                                       # 查看当前 Hugo 版本
```
