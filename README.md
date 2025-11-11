# test.github.io：R1 初始化与最小 GitHub Pages

> 本仓库位于十轮演示中的第 R1 轮，目标是用最小配置跑通 GitHub Pages + GitHub Actions 的自动化发布。

## 目录
- [0) 项目简介与目标](#0-项目简介与目标)
- [1) 什么是 CI/CD 与 GitHub Pages 的关系](#1-什么是-cicd-与-github-pages-的关系)
- [2) 逐步操作指引：从新建仓库到网页可访问](#2-逐步操作指引从新建仓库到网页可访问)
- [3) 工作流文件详解](#3-工作流文件详解)
- [4) 验收与自测](#4-验收与自测)
- [5) 常见问题与排错](#5-常见问题与排错)
- [6) 后续路线图（预告）](#6-后续路线图预告)

## 0) 项目简介与目标
- 本仓库用于演示如何仅凭 GitHub Pages 即可托管一个最小静态网站。
- 当前处于十轮演示的 R1 阶段，未来轮次将逐步扩展到通用构建能力与更多 CI/CD 功能。
- 新建文件与用途概览：
  | 文件路径 | 作用说明 |
  | --- | --- |
  | `index.html` | 最小静态页面，展示项目名、分支占位、构建时间占位，并链接 README |
  | `.gitignore` | 忽略常见本地临时目录与日志 |
  | `.github/workflows/pages.yml` | GitHub Actions 工作流，构建并部署 Pages |
  | `README.md` | 中文说明文档，指导从创建仓库到部署完成 |

## 1) 什么是 CI/CD 与 GitHub Pages 的关系
- CI/CD（持续集成与持续交付）旨在让代码变更持续地被集成、测试与发布，从而减少手动步骤并提升交付频率。
- 在本仓库中，CI/CD 链路是：推送到 `main` 分支或手动触发 → 工作流复制 `index.html` 到 `dist/` 并写入构建时间 → 上传为 Pages 工件 → 自动部署到 GitHub Pages。
- 最小权限原则要求仅授予动作所需的最小令牌权限，因此 `pages: write` 用于发布页面，`id-token: write` 用于在部署时与 Pages 服务安全握手。

## 2) 逐步操作指引：从新建仓库到网页可访问
以下步骤适合零基础用户在 GitHub 网页端完成。

### 步骤 A：创建仓库
1. 登录 GitHub，点击页面右上角的 **New**。
2. 在 **Repository name** 输入框填入 `test.github.io`。
3. 可不勾选 **Add a README file**（本项目会提供完整 README）。
4. 点击 **Create repository** 完成创建。

### 步骤 B：推送或上传本项目文件
- 方式 1：网页上传
  1. 打开仓库主页，点击 **Add file** → **Upload files**。
  2. 将本项目的 `index.html`、`.gitignore`、`.github/workflows/pages.yml`、`README.md` 拖拽到上传区域。
  3. 页面下方填写提交说明，点击 **Commit changes**。
- 方式 2：命令行推送
  ```bash
  git clone https://github.com/<你的用户名>/test.github.io.git
  cd test.github.io
  # 将上述文件复制到本地仓库后执行：
  git add .
  git commit -m "Initialize GitHub Pages R1"
  git push origin main
  ```

### 步骤 C：启用 GitHub Pages
1. 打开仓库的 **Settings**。
2. 在左侧导航选择 **Pages**。
3. 在 **Build and deployment** 的 **Source** 下拉框中选择 **GitHub Actions**。
4. 点击 **Save**。
   - 【需要截图】Settings → Pages 中设置 Source: GitHub Actions 的位置。

### 步骤 D：查看 Actions 运行
1. 切换到仓库的 **Actions** 标签页。
2. 打开最近一次名为 **Deploy to GitHub Pages** 的运行记录。
3. 展开 `build` 与 `deploy` 两个 Job，确认所有步骤均显示成功。
   - 【需要截图】Actions 运行详情页显示成功状态的画面。

### 步骤 E：访问网站
1. 返回 **Settings → Pages** 页面，在 **Website** 区域查看已发布的网址。
2. 若仓库名为 `<用户名>.github.io`，访问 `https://<用户名>.github.io/`；否则访问 `https://<用户名>.github.io/test.github.io/`。

## 3) 工作流文件详解
- `on` 触发器：监听 `main` 分支的 `push` 事件，并允许通过 **Run workflow** 手动触发，方便在无代码变更时重新部署。
- `permissions` 字段：
  - `contents: read` 让工作流读取仓库内容。
  - `pages: write` 允许部署到 GitHub Pages。
  - `id-token: write` 使部署步骤能获取短期令牌，完成与 Pages 的 OIDC 鉴权。
- `concurrency`：设置部署分组为 `pages`，确保新的部署会取消同分组的旧运行，避免重复发布。
- `actions/checkout@v4`：检出当前提交，保证后续步骤可以访问 `index.html` 等文件。
- 构建步骤中把页面复制到 `dist/` 并追加 `<!-- built at ... -->` 注释，记录 UTC 构建时间。
- `actions/upload-pages-artifact@v3`：将 `dist/` 打包成 Pages 产物，供后续部署步骤使用。
- `actions/deploy-pages@v4`：把刚才上传的工件发布到 GitHub Pages，并在日志中提供最终访问链接。

## 4) 验收与自测
- 成功标准：
  1. Actions 工作流运行通过。
  2. Pages 页面可访问，显示 “Hello, GitHub Pages”。
  3. 在浏览器查看源代码，页尾可见形如 `<!-- built at 2025-01-01T00:00:00Z -->` 的注释。
- 最小变更测试：将 `index.html` 的 `<title>` 修改为其他文字并推送，确认工作流再次运行并发布更新。

## 5) 常见问题与排错
- Pages 链接未生成：确认已在 Settings → Pages 中选择 GitHub Actions 作为 Source，并等待首次部署完成。
- 工作流权限不足：确保未额外收紧工作流权限，或者在组织策略中允许 `pages: write` 与 `id-token: write`。
- 访问缓慢：若网络环境不稳定，可尝试使用 CDN 加速或等待稳定时段访问。
- 脚本不可执行：若在 Windows 编辑后上传，确保工作流脚本仍保持 LF 换行，避免 `run` 步骤因换行符导致的解释错误。

## 6) 后续路线图（预告）
- R2：引入 `ci/build.sh`，自动识别常见静态站点构建工具。
- R3：加入 Pull Request 校验与工件预览。
- R4：支持 Pull Request 临时预览链接。
- R5：新增链接检查等质量门禁。
- R9、R10：扩展至 Release 与 VPS 部署演示。
