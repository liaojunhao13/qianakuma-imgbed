# 恶魔茜图床

部署于 Cloudflare 的个人 Serverless 图床，使用 Workers 提供前端与 API、R2 存储图片、D1 存储索引和配置。

- 站点：<https://img.qianakuma.xyz>
- 仓库：<https://github.com/liaojunhao13/qianakuma-imgbed>
- 所有者：恶魔茜 / QianAkuma

## 架构与资源

| 资源 | 名称 | Binding | 用途 |
| --- | --- | --- | --- |
| Worker | `qianakuma-imgbed` | - | 前端、上传 API、图片访问 |
| D1 | `qianakuma-imgbed-db` | `img_d1` | 元数据、设置、认证信息 |
| R2 | `qianakuma-imgbed-r2` | `img_r2` | 图片文件 |
| Custom Domain | `img.qianakuma.xyz` | - | 图床独立域名 |

图床只使用 `img.qianakuma.xyz`，不会修改或覆盖 `blog.qianakuma.xyz`。

## 本地准备

```bash
npm ci
npx wrangler login
```

创建并初始化 Cloudflare 资源：

```bash
npx wrangler d1 create qianakuma-imgbed-db
npx wrangler d1 execute qianakuma-imgbed-db --remote --file database/init.sql
npx wrangler r2 bucket create qianakuma-imgbed-r2
```

D1 和 R2 的 binding 必须分别为 `img_d1` 和 `img_r2`。公开的资源名称和 D1 ID 配置在 `deploy/worker/wrangler.toml`。

## Secrets

以下敏感信息必须设置为 Cloudflare Worker Secrets，不得提交到 GitHub：

```bash
npx wrangler secret put BASIC_USER --config deploy/worker/wrangler.toml
npx wrangler secret put BASIC_PASS --config deploy/worker/wrangler.toml
npx wrangler secret put AUTH_CODE --config deploy/worker/wrangler.toml
npx wrangler secret put RESET_KEY --config deploy/worker/wrangler.toml
```

| Secret | 用途 |
| --- | --- |
| `BASIC_USER` | 管理员用户名 |
| `BASIC_PASS` | 管理员强密码 |
| `AUTH_CODE` | 访客上传和访问认证码 |
| `RESET_KEY` | 紧急重置认证配置，使用后应轮换 |

R2 通过 Worker binding 访问，应用本身不需要 R2 Access Key 或 Secret Access Key。只有外部 S3 客户端需要单独创建 R2 API Token，并且同样不得提交到仓库。

## 部署

```bash
node deploy/worker/generate-routes.js
npx wrangler deploy --config deploy/worker/wrangler.toml
```

`deploy/worker/wrangler.toml` 使用精确 Custom Domain：

```toml
routes = [
  { pattern = "img.qianakuma.xyz", custom_domain = true }
]
```

Cloudflare 会为该 Worker 关联 DNS 和 TLS 证书。不要修改 `blog.qianakuma.xyz` 的 DNS 或 Worker Route。

## GitHub Actions

自动部署工作流位于 `.github/workflows/deploy-worker.yml`。需要在 GitHub Actions Secrets 中配置：

- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`
- `D1_DATABASE_ID`
- `R2_BUCKET_NAME`
- `WORKER_NAME`，值为 `qianakuma-imgbed`

管理员密码和上传码只保存在 Cloudflare Worker Secrets，不要放进 `WORKER_VARS`。

## Markdown 引用

普通文章图片：

```md
![WMMA fragment 示意图](https://img.qianakuma.xyz/file/cuda/wmma-fragment.png)
```

Astro 文章封面：

```yaml
image: "https://img.qianakuma.xyz/file/cuda/wmma-fragment.png"
```

建议文件名只使用英文字母、数字和连字符，目录保持简短稳定。

## 上游与许可

本仓库基于开源项目 [CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed) 定制。源码遵循原项目 MIT License，原始版权声明保留在 `LICENSE` 中。
