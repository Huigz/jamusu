# GitHub + Cloudflare Workers 自动部署指南

本指南适用于 Hugo 站点仓库 `Huigz/jamusu`。Cloudflare 已将静态资源部署整合到 Workers；配置完成后，每次向 GitHub `main` 分支 push 都会自动构建并部署站点。

## 1. 仓库配置

`hugo.toml` 中的正式站点地址应为：

```toml
baseURL = "https://about.sujiamu.com"
```

仓库根目录的 `wrangler.jsonc` 用于声明 Worker 名称、兼容日期和 Hugo 输出目录：

```json
{
  "name": "jamusu",
  "compatibility_date": "2026-06-29",
  "assets": {
    "directory": "./public",
    "not_found_handling": "404-page"
  }
}
```

`404-page` 会让找不到的路径返回 Hugo 生成的 `404.html`和正确的 HTTP 404 状态。

## 2. 连接 GitHub

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 进入 **Workers & Pages** → **Create application** → **Import a repository** / **Connect to Git**。
3. 授权 **Cloudflare Workers & Pages** GitHub App。
4. 建议选择 **Only select repositories**，只授权 `Huigz/jamusu`。
5. 选择 `Huigz/jamusu`，生产分支选择 `main`。

## 3. 填写 Workers Builds 表单

| 选项 | 值 |
| --- | --- |
| Project name | `jamusu` |
| Build command | `hugo --gc --minify` |
| Deploy command | `npx wrangler deploy` |
| Root directory | 留空（仓库根目录） |
| Builds for non-production branches | 可先关闭 |

`hugo --gc --minify` 先将站点生成到 `public/`，然后 `wrangler deploy` 读取 `wrangler.jsonc` 并将 `public/` 作为 Workers Static Assets 上传。

Cloudflare 当前构建镜像自带 Hugo Extended。如果需要与本地环境严格一致，可在 **Settings → Build → Build variables and secrets** 添加：

| 变量 | 值 |
| --- | --- |
| `HUGO_VERSION` | `0.124.1` |

这不是本次 Wrangler 报错的原因；日志已显示 Hugo 0.147.7 构建成功。

## 4. 绑定子域名

Worker 首次部署成功后：

1. 进入 `jamusu` Worker。
2. 打开 **Settings → Domains & Routes**。
3. 选择 **Add → Custom Domain**。
4. 输入 `about.sujiamu.com` 并确认。

如果 `sujiamu.com` 已在同一 Cloudflare 账户中托管，Cloudflare 会自动创建所需 DNS 记录和 HTTPS 证书。如果该子域名已有 A、AAAA 或 CNAME 记录，先处理冲突记录。

## 5. 自动部署流程

以后只需要：

```bash
git add <changed-files>
git commit -m "Describe the change"
git push origin main
```

Workers Builds 会依次执行：

```text
hugo --gc --minify
npx wrangler deploy
```

在 GitHub commit check 和 Cloudflare Worker 的 **Deployments** 页面可查看结果。

## 6. 常见问题

- **`A compatibility_date is required`**：确认 `wrangler.jsonc` 已提交到 GitHub，且 Cloudflare 的 Root directory 为仓库根目录。
- **Worker name 显示 `undefined`**：确认 `wrangler.jsonc` 中存在 `"name": "jamusu"`。
- **Theme not found**：检查 `.gitmodules` 是否同时包含主题的 `path` 和公开 `url`。
- **站内绝对链接指向旧域名**：确认 `hugo.toml` 的 `baseURL` 为 `https://about.sujiamu.com`。
- **自定义域名无法添加**：检查 DNS 中是否有同名 A、AAAA 或 CNAME 记录。

## 官方参考

- [Cloudflare Workers Builds configuration](https://developers.cloudflare.com/workers/ci-cd/builds/configuration/)
- [Cloudflare Workers Git integration](https://developers.cloudflare.com/workers/ci-cd/builds/git-integration/)
- [Cloudflare Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/)
- [Cloudflare Workers Custom Domains](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/)
- [Cloudflare Workers build image](https://developers.cloudflare.com/workers/ci-cd/builds/build-image/)
