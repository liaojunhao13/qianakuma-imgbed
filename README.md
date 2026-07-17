# QianAkuma ImgBed

A personal serverless image host running on Cloudflare Workers, R2, and D1.

- Site: <https://img.qianakuma.xyz>
- Repository: <https://github.com/liaojunhao13/qianakuma-imgbed>
- Owner: QianAkuma / 恶魔茜
- Chinese deployment guide: [README_zh.md](README_zh.md)

## Resources

| Resource | Name | Binding |
| --- | --- | --- |
| Worker | `qianakuma-imgbed` | - |
| D1 | `qianakuma-imgbed-db` | `img_d1` |
| R2 | `qianakuma-imgbed-r2` | `img_r2` |
| Custom domain | `img.qianakuma.xyz` | - |

Runtime credentials must be stored as Cloudflare Worker Secrets and must never be committed:

```bash
npx wrangler secret put BASIC_USER --config deploy/worker/wrangler.toml
npx wrangler secret put BASIC_PASS --config deploy/worker/wrangler.toml
npx wrangler secret put AUTH_CODE --config deploy/worker/wrangler.toml
npx wrangler secret put RESET_KEY --config deploy/worker/wrangler.toml
```

Deploy with:

```bash
npm ci
node deploy/worker/generate-routes.js
npx wrangler deploy --config deploy/worker/wrangler.toml
```

See [README_zh.md](README_zh.md) for D1/R2 setup, GitHub Actions, DNS, security, and Markdown usage.

## Upstream And License

This repository is customized from [CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed). It remains licensed under the MIT License, with the original copyright notice retained in `LICENSE`.
