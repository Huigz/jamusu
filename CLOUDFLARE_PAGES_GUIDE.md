# GitHub + Cloudflare Pages 自动部署指南

本指南适用于当前 Hugo 站点仓库：`Huigz/jamusu`。配置完成后，每次向 GitHub `main` 分支推送提交，Cloudflare Pages 都会自动构建并发布站点。

## 1. 确定子域名

先确定最终访问地址，例如：

```text
https://cv.example.com/
```

将 `hugo.toml` 中的 `baseURL` 改为这个完整地址：

```toml
baseURL = "https://cv.example.com/"
```

当前仓库的 `baseURL` 是 `https://jamusu.com`。如果实际使用其他子域名，应在第一次正式部署前修改，否则 RSS、Open Graph 等绝对链接可能指向错误地址。

## 2. 将 GitHub 仓库连接到 Cloudflare Pages

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 进入 **Workers & Pages** → **Create application** → **Pages** → **Connect to Git** / **Import an existing Git repository**。
3. 选择 GitHub，安装并授权 **Cloudflare Workers & Pages** GitHub App。
4. 建议选择 **Only select repositories**，只授权 `Huigz/jamusu`。
5. 选择 `Huigz/jamusu` 并开始设置。

Cloudflare Pages 的 Git 集成会在每次 push 后自动构建；非生产分支和 Pull Request 还可生成独立预览地址。

## 3. 设置 Hugo 构建参数

在 **Set up builds and deployments** 页面使用：

| 选项 | 值 |
| --- | --- |
| Production branch | `main` |
| Framework preset | `Hugo` |
| Build command | `hugo --gc --minify` |
| Build output directory | `public` |
| Root directory | 留空（仓库根目录） |

在 **Environment variables** 中添加：

| 变量 | 值 |
| --- | --- |
| `HUGO_VERSION` | `0.124.1` |

如果启用 Preview deployments，在 **Production** 和 **Preview** 两个环境都添加 `HUGO_VERSION`。当前站点已在 Hugo 0.124.1 下验证。

主题 `themes/hugo-bearblog` 是 Git submodule。当前 `.gitmodules` 同时包含 `path` 和公开 `url`，可供 Cloudflare 克隆。

点击 **Save and Deploy**。首次成功后会获得一个类似下面的默认地址：

```text
https://<project-name>.pages.dev
```

## 4. 绑定 Cloudflare 子域名

1. 进入 **Workers & Pages** → 选择刚创建的 Pages 项目。
2. 打开 **Custom domains** → **Set up a domain**。
3. 输入完整子域名，例如 `cv.example.com`。
4. 点击 **Continue** / **Activate domain**。

如果主域名已在同一 Cloudflare 账户内托管 DNS，Cloudflare 会自动创建 CNAME 记录。

如果 DNS 在其他服务商，必须在 Pages 项目中添加 Custom domain **之后**，再在 DNS 服务商处添加：

| Type | Name | Target |
| --- | --- | --- |
| `CNAME` | `cv` | `<project-name>.pages.dev` |

不要只手动添加 CNAME 而跳过 Pages 的 **Custom domains** 流程；Cloudflare 官方文档说明这会导致 `522` 错误。

## 5. 验证自动部署

以后的更新流程是：

```bash
git add <changed-files>
git commit -m "Describe the change"
git push origin main
```

push 后检查：

1. GitHub 该 commit 旁的 Cloudflare check 是否通过。
2. Cloudflare Pages 项目的 **Deployments** 中是否出现新的 Production deployment。
3. 子域名是否显示最新内容，HTTPS 证书状态是否为 Active。

## 6. 常见问题

- **Hugo 命令不可用或版本不一致**：确认 `HUGO_VERSION=0.124.1` 已设置到正确环境。
- **Theme not found**：检查 Cloudflare 克隆日志，并确认 `.gitmodules` 中的 `path` 和 `url` 未被删除。
- **页面能打开，但图片、RSS 或分享链接域名错误**：检查 `hugo.toml` 的 `baseURL`。
- **子域名返回 522**：先在 Pages 的 **Custom domains** 中添加子域名，再检查 CNAME。
- **不想某次 push 触发构建**：可在 commit message 前加 `[CF-Pages-Skip]`。

## 官方参考

- [Cloudflare Pages: Deploy a Hugo site](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/)
- [Cloudflare Pages: Git integration](https://developers.cloudflare.com/pages/configuration/git-integration/)
- [Cloudflare Pages: GitHub integration](https://developers.cloudflare.com/pages/configuration/git-integration/github-integration/)
- [Cloudflare Pages: Custom domains](https://developers.cloudflare.com/pages/configuration/custom-domains/)
- [Cloudflare Pages: Debugging Pages](https://developers.cloudflare.com/pages/configuration/debugging-pages/)
