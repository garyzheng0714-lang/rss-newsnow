# rss-newsnow

![类型：资讯工具](https://img.shields.io/badge/%E7%B1%BB%E5%9E%8B-%E8%B5%84%E8%AE%AF%E5%B7%A5%E5%85%B7-0f766e)
![技术栈：React / Nitro](https://img.shields.io/badge/%E6%8A%80%E6%9C%AF%E6%A0%88-React%20%2F%20Nitro-61dafb)
![状态：维护中](https://img.shields.io/badge/%E7%8A%B6%E6%80%81-%E7%BB%B4%E6%8A%A4%E4%B8%AD-2ea44f)
![README：中文](https://img.shields.io/badge/README-%E4%B8%AD%E6%96%87-d73a49)

`rss-newsnow` 是一个基于 React 与 Nitro 的实时热点阅读应用，用于聚合多来源热榜、资讯源和可选 MCP 访问入口。

## 仓库定位

- 分类：资讯工具 / 热榜阅读器。
- 面向对象：需要统一浏览多平台实时热点、维护来源列表或把资讯源接入 agent 工作流的开发者。
- 使用边界：本仓库是 NewsNow 应用代码与部署配置，不是 RSS 订阅服务，也不是通用爬虫框架；数据来源主要由 `shared/sources.ts`、`shared/sources.json` 和 `server/sources/` 中的实现决定。

## 功能概览

- 提供实时与热门新闻阅读界面。
- 从 `shared/sources.ts` 和相关 metadata 文件生成来源列表。
- 支持 GitHub OAuth 登录与用户数据同步。
- 支持服务端缓存，默认缓存时间为 `30` 分钟。
- 支持自适应抓取间隔，每个来源最小间隔为 `2` 分钟。
- 提供 MCP server API，便于 agent 读取已部署实例。
- 包含 PWA 图标、service worker 和可安装应用相关资源。
- 包含 Docker、Cloudflare Pages 与 Vercel 相关配置文件。

> 说明：当前 demo 版本主要聚焦中文内容；现有路线图中提到更多语言和自定义能力会在后续版本扩展。

## 技术栈

- React `19`
- TypeScript
- Vite `6`
- Nitro via `vite-plugin-with-nitro`
- TanStack Router 与 React Query
- Jotai
- UnoCSS
- db0 database connector
- Vitest 与 ESLint
- Wrangler / Cloudflare Pages 工作流

## 项目结构

```text
.
├── src/                 # React routes、components、hooks、atoms 和样式
├── server/              # Nitro API、数据 getter、MCP route 和工具函数
├── shared/              # 来源定义、metadata、常量和共享类型
├── scripts/             # 来源与 favicon 生成脚本
├── public/              # 图标、PWA、字体、sitemap 和 service worker
├── test/                # Vitest 测试
├── Dockerfile
├── docker-compose.yml
├── docker-compose.local.yml
├── example.env.server
├── example.wrangler.toml
└── package.json
```

## 快速开始

需要 Node.js `>= 20` 和 pnpm。仓库通过 `packageManager` 固定 pnpm 版本。

```bash
corepack enable
pnpm install
pnpm dev
```

`pnpm dev` 会先执行来源与 favicon 预生成，再启动 Vite/Nitro 开发服务。

## 常用命令

| Command | 说明 |
| --- | --- |
| `pnpm dev` | 生成来源资源并启动本地 Vite/Nitro 开发服务。 |
| `pnpm build` | 生成来源资源并构建应用。 |
| `pnpm start` | 使用 `.env.server` 运行构建后的 Nitro server。 |
| `pnpm preview` | 为 Cloudflare Pages 构建并运行 `wrangler pages dev`。 |
| `pnpm deploy` | 构建并通过 Wrangler 部署 public output。 |
| `pnpm test` | 运行 Vitest。 |
| `pnpm typecheck` | 对 node 与 app TypeScript 配置运行类型检查。 |
| `pnpm lint` | 运行 ESLint。 |
| `pnpm log` | 查看 Cloudflare Pages 项目 `newsnow` 的部署日志。 |

## 配置

复制 `example.env.server` 为 `.env.server` 后按需填写：

```env
G_CLIENT_ID=
G_CLIENT_SECRET=
JWT_SECRET=
INIT_TABLE=true
ENABLE_CACHE=true
PRODUCTHUNT_API_TOKEN=
```

配置说明：

- `G_CLIENT_ID` 和 `G_CLIENT_SECRET` 用于 GitHub OAuth。
- `JWT_SECRET` 用于签名用户 session。
- `INIT_TABLE=true` 会在首次运行时初始化数据库表；初始化后可按需关闭。
- `ENABLE_CACHE=true` 开启服务端缓存。
- `PRODUCTHUNT_API_TOKEN` 仅在需要 Product Hunt 数据时填写。

数据库连接遵循 db0 connector 生态。仓库中现有 Cloudflare 配置以 D1 为推荐路径，可从 `example.wrangler.toml` 开始调整。

## 部署

### Cloudflare Pages

- Build command：`pnpm run build`
- Output directory：`dist/output/public`
- D1 binding 配置可参考 `example.wrangler.toml`。

### Docker 部署

使用 `docker-compose.yml` 中的镜像配置：

```bash
docker compose up
```

如需本地构建镜像：

```bash
docker compose -f docker-compose.local.yml up --build
```

### GitHub OAuth

1. 创建 GitHub OAuth App。
2. 将 callback URL 设置为 `https://your-domain.example/api/oauth/github`。
3. 在 `.env.server` 或托管平台中配置 client ID 和 secret。

## MCP 服务接入

已部署的 NewsNow 实例可通过 MCP client 访问，示例配置如下：

```json
{
  "mcpServers": {
    "newsnow": {
      "command": "npx",
      "args": ["-y", "newsnow-mcp-server"],
      "env": {
        "BASE_URL": "https://your-newsnow-domain.example"
      }
    }
  }
}
```

`BASE_URL` 应指向你实际部署的 NewsNow 实例。

## 添加数据源

来源定义主要位于 `shared/sources.ts`、`shared/sources.json` 和相关 `server/sources/` getter 代码。修改来源后，通过常规 `dev` 或 `build` 命令重新生成来源资源。

贡献流程可参考 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 注意事项

- 根 README 以中文维护；仓库中仍保留 [README.zh-CN.md](README.zh-CN.md) 和 [README.ja-JP.md](README.ja-JP.md) 作为历史多语言文档。
- 本仓库的 `package.json` 标记为 `private: true`，不要把 README 中的命令理解为 npm package 发布流程。

## 许可证

[MIT](./LICENSE) © ourongxing
