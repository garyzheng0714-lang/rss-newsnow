![](/public/og-image.png)

English | [简体中文](README.zh-CN.md) | [日本語](README.ja-JP.md)

# NewsNow

NewsNow is a React and Nitro web app for reading real-time and trending news from many configured sources. It includes a clean reading interface, source metadata generation, optional GitHub OAuth login, optional caching and sync, and an MCP server endpoint for agent integrations.

> [!NOTE]
> This demo version currently focuses on Chinese-language content. The existing roadmap calls out broader language support and more customization in a later version.

## Features

- Real-time and hot-news reading interface.
- Source list generated from `shared/sources.ts` and related metadata files.
- GitHub OAuth login with user data synchronization.
- Optional server-side cache, with a 30-minute default cache duration.
- Adaptive scraping intervals with a minimum 2-minute interval per source.
- MCP server support through the server API.
- PWA assets and service-worker files for installable browser usage.
- Docker, Cloudflare Pages, and Vercel-oriented deployment files.

## Tech Stack

- React 19
- TypeScript
- Vite 6
- Nitro via `vite-plugin-with-nitro`
- TanStack Router and React Query
- Jotai state management
- UnoCSS
- db0 database connector support
- Vitest and ESLint
- Wrangler for Cloudflare Pages workflows

## Project Structure

```text
.
├── src/                 # React app routes, components, hooks, atoms, and styles
├── server/              # Nitro server APIs, data getters, MCP route, and utilities
├── shared/              # Source definitions, metadata, constants, and shared types
├── scripts/             # Source and favicon generation scripts
├── public/              # Icons, PWA assets, fonts, sitemap, and service worker
├── test/                # Vitest tests
├── Dockerfile
├── docker-compose.yml
├── docker-compose.local.yml
├── example.env.server
├── example.wrangler.toml
└── package.json
```

## Getting Started

Requires Node.js `>= 20` and pnpm. The repository pins pnpm through `packageManager`.

```bash
corepack enable
pnpm install
pnpm dev
```

The development script runs the favicon/source pre-generation step before starting Vite.

## Scripts

| Command | Description |
| --- | --- |
| `pnpm dev` | Generate source assets and start the local Vite/Nitro dev server. |
| `pnpm build` | Generate source assets and build the app. |
| `pnpm start` | Run the built Nitro server with `.env.server`. |
| `pnpm preview` | Build for Cloudflare Pages and run `wrangler pages dev`. |
| `pnpm deploy` | Build and deploy the public output with Wrangler. |
| `pnpm test` | Run Vitest. |
| `pnpm typecheck` | Run TypeScript checks for node and app configs. |
| `pnpm lint` | Run ESLint. |
| `pnpm log` | Tail Cloudflare Pages deployment logs for the `newsnow` project name. |

## Configuration

Copy `example.env.server` to `.env.server` for local server runs:

```env
G_CLIENT_ID=
G_CLIENT_SECRET=
JWT_SECRET=
INIT_TABLE=true
ENABLE_CACHE=true
PRODUCTHUNT_API_TOKEN=
```

Configuration notes:

- `G_CLIENT_ID` and `G_CLIENT_SECRET` are used for GitHub OAuth.
- `JWT_SECRET` signs user sessions.
- `INIT_TABLE=true` initializes database tables on first run; turn it off after initialization if desired.
- `ENABLE_CACHE=true` enables server-side cache behavior.
- `PRODUCTHUNT_API_TOKEN` is optional and only needed for Product Hunt data.

Database connectors follow the db0 connector ecosystem. Cloudflare D1 is the recommended path in the existing deployment docs/configuration; start from `example.wrangler.toml` when using Cloudflare.

## Deployment

### Cloudflare Pages

- Build command: `pnpm run build`
- Output directory: `dist/output/public`
- Use `example.wrangler.toml` as a starting point for D1 binding configuration.

### Docker

Use the published image configuration in `docker-compose.yml`:

```bash
docker compose up
```

For local image builds, use `docker-compose.local.yml`:

```bash
docker compose -f docker-compose.local.yml up --build
```

### GitHub OAuth

1. Create a GitHub OAuth app.
2. Set the callback URL to `https://your-domain.example/api/oauth/github`.
3. Configure the client ID and secret in `.env.server` or the hosting platform.

## MCP Server

NewsNow can be exposed to MCP clients through the published MCP package:

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

Set `BASE_URL` to the deployed NewsNow instance you want the MCP client to use.

## Adding Data Sources

Source definitions live primarily in `shared/sources.ts`, `shared/sources.json`, and related server getter code. Run the source generation script through the normal `dev` or `build` command after changing sources.

For contribution details, see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](./LICENSE) © ourongxing
