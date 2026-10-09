# Getting started, project structure, configuration, identity, scopes

Inspected against `@whatalo/plugin-sdk` 1.5.0 and `whatalo`/`create-whatalo-plugin` 1.6.0; summary only, read the linked pages for full contracts. Sources: [source-map.md](source-map.md).

## What a plugin is

A web app you host yourself, loaded by the Whatalo admin in a sandboxed iframe as one or more sidebar pages. It talks to the admin through the App Bridge (`postMessage`, in `@whatalo/plugin-sdk/bridge`) and to store data through the Data Bridge or your backend's `WhataloClient`.
Sources: [Plugin SDK](https://developers.whatalo.com/docs/plugin-sdk), [Platform Overview](https://developers.whatalo.com/docs/plugin-sdk/getting-started/platform-overview), [Plugin Architecture](https://developers.whatalo.com/docs/plugin-sdk/getting-started/plugin-architecture).

## Prerequisites

- A Whatalo developer account (separate from a store owner account) and at least one development store; the docs allow up to 3 per developer account.
- Node.js: the docs and `whatalo` package state Node.js 18+. The starter depends on Vite 7; the [vite@7 npm registry metadata](https://registry.npmjs.org/vite/7) declares `engines.node` `^20.19.0 || >=22.12.0`. Use a Node.js version that satisfies the starter's dependencies.
- pnpm (the starter's `whatalo.app.toml` uses `pnpm dev` and `pnpm build`).
- No manual tunnel install: `whatalo dev` downloads and manages its tunnel tool on first use.

Sources: [Prerequisites](https://developers.whatalo.com/docs/plugin-sdk/getting-started/prerequisites), [Quick Start](https://developers.whatalo.com/docs/plugin-sdk/quick-start).

## First run

Portal URL resolution in the inspected `@whatalo/cli` 1.6.0 bundle differs by command:

| Commands | Order |
| --- | --- |
| `login`, `dev` | `--portal-url` → `[dev] portal_url` in `whatalo.app.toml` (`dev`) → `WHATALO_DEVELOPER_PORTAL_URL` → `https://developers.whatalo.com` |
| `init` | `--portal-url` → portal stored in the login session → `WHATALO_DEVELOPER_PORTAL_URL` → `https://developers.whatalo.com` |
| Other authenticated commands | Portal stored in the login session |

The [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview#portal-url-resolution) documents one global order with a localhost default (see Source conflicts). Pass `--portal-url` only where the command's `--help` lists it:

```sh
pnpm add -g whatalo
whatalo login --portal-url https://developers.whatalo.com
whatalo init --portal-url https://developers.whatalo.com
cd my-plugin
whatalo dev --portal-url https://developers.whatalo.com
```

`whatalo init` registers (or links) the plugin, scaffolds a slug-named directory, installs dependencies, and shows the client secret **once** for a new plugin. Save it into `.env` as `WHATALO_CLIENT_SECRET`.
Sources: [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview), [whatalo init](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/init).

## Project structure (starter `react-vite`)

```text
my-plugin/
├── whatalo.app.ts        # manifest: defineApp()
├── whatalo.app.toml      # local CLI config
├── vite.config.ts        # proxies /api and /webhooks/whatalo to the backend
├── server/
│   ├── auth.ts           # requireSession: verifyWhataloSessionToken(..., { expectedAppId: app.id })
│   ├── env.ts            # zod-parsed WHATALO_CLIENT_SECRET, WHATALO_WEBHOOK_SECRET, SERVER_PORT
│   └── index.ts          # Express: /api/status and POST /webhooks/whatalo
├── src/
│   ├── main.tsx          # imports ./theme-bootstrap first
│   ├── theme-bootstrap.ts
│   ├── app.tsx           # useThemeSync + useAutoResize + switch(currentPage)
│   ├── lib/plugin-api.ts # pluginFetch: session token + one 401 retry
│   ├── pages/settings.tsx
│   ├── webhooks/verify.ts
│   └── components/whatalo-ui/
└── .env.example          # WHATALO_CLIENT_ID, WHATALO_CLIENT_SECRET, WHATALO_WEBHOOK_SECRET, SERVER_PORT
```

`pnpm dev` runs Vite (port 5173) and the Express backend (port 8787) together; Vite proxies `/api` and `/webhooks/whatalo` so the tunnel exposes one origin. Scripts: `dev`, `build`, `preview`, `type-check`.
Sources: `create-whatalo-plugin@1.6.0` template; [Quick Start](https://developers.whatalo.com/docs/plugin-sdk/quick-start#what-was-scaffolded); [Project Configuration](https://developers.whatalo.com/docs/plugin-sdk/configuration/project-config).

## Plugin identity

| Value | Example | Use |
| --- | --- | --- |
| Manifest `id` | `my-plugin` | `appId`/`aud` in session tokens, `claims.appId` in connection state, `expectedAppId` |
| `WHATALO_CLIENT_ID` | `a6-XXXXXXXX` | Public plugin ID for the portal and CLI; `plugin_id` in TOML |
| `WHATALO_CLIENT_SECRET` | `sk_...` | Verifies session tokens and connection state, signs connection callbacks |

Import the manifest and compare with `app.id`; comparing with `WHATALO_CLIENT_ID` rejects every valid token. The manifest `id` is immutable after registration.
Sources: [Plugin identifiers](https://developers.whatalo.com/docs/plugin-sdk#plugin-identifiers), [Plugin Manifest](https://developers.whatalo.com/docs/plugin-sdk/configuration/plugin-manifest#the-id-field-is-immutable).

## Manifest: `whatalo.app.ts`

Import `defineApp` from `@whatalo/plugin-sdk/manifest`. It collects all errors and throws `ManifestValidationError` (`errors: { field, message }[]`). The CLI validates it before `dev`, `deploy`, and `validate`.

Key constraints (full field table: [Plugin Manifest](https://developers.whatalo.com/docs/plugin-sdk/configuration/plugin-manifest#required-fields)):

- `id` is lowercase with hyphens, 3–50 chars, and immutable after registration.
- `shortDescription` (10–160 chars) is required for new manifests and publication; publication requires `description` ≥100 chars.
- `version` is `major.minor.patch`; `appUrl` is absolute HTTPS without credentials (HTTP only for loopback); `permissions` needs at least one scope.

Optional: `author.url`, `webhookUrl` (omit when unused; do not assign `""`), `authUrl` (see [app-bridge.md](app-bridge.md#vendor-account-connection)), `callbackUrl` (reserved, unused — do not set), `adminUI.pages`, `webhooks: { event, description? }[]`.

`adminUI.pages` entries: `path` (single local segment, unique ignoring a leading `/`), `title` (non-blank), `icon`, `position` (`"sidebar"` or `"settings"`), optional non-blank `navigationLabel`. The first page is the default. `path` is the value of `currentPage`. At least one page is required for submission and private publication.
Sources: [Plugin Manifest](https://developers.whatalo.com/docs/plugin-sdk/configuration/plugin-manifest), [Navigation](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/navigation).

## Project config: `whatalo.app.toml`

Required: `[plugin] name, plugin_id, slug`; `[build] dev_command, build_command, output_dir`; `[dev] port` (1024–65535, default 5173). Optional: `[webhooks] url, events`, `[dev] store`. It holds no secrets, is committed, and is never sent to the platform at runtime. `dev_command` must serve the iframe, `/api/*`, and `/webhooks/whatalo` on one local origin.
Source: [Project Configuration](https://developers.whatalo.com/docs/plugin-sdk/configuration/project-config).

## Environment variables

Key points (full list: [Environment Variables](https://developers.whatalo.com/docs/plugin-sdk/configuration/environment-variables)):

- `WHATALO_CLIENT_SECRET` is shown once by `whatalo init`; `whatalo env pull` does not return it.
- `WHATALO_API_KEY` is a store-scoped test key written by `whatalo dev`; `env pull` does not return it.
- `WHATALO_WEBHOOK_SECRET` is your own value for local test signatures (starter backend and `whatalo webhook trigger`).

Never commit `.env`; the starter `.gitignore` excludes `.env`, `.env.local`, and `.whatalo/`. `whatalo env show` masks secrets.
Sources: [Environment Variables](https://developers.whatalo.com/docs/plugin-sdk/configuration/environment-variables), [whatalo env](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/env), [whatalo webhook trigger](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/webhook-trigger).

## Scopes

Scope names follow `read:<resource>` / `write:<resource>`. Use the scope-to-method table in [Scopes & Permissions](https://developers.whatalo.com/docs/plugin-sdk/configuration/scopes-and-permissions#all-available-scopes); `read:analytics` is documented as "Coming soon".

Rules: declare at least one scope and only what you use; a `write:*` scope does **not** grant the matching `read:*`; tokens without a scope get `403`. Adding scopes in an update requires merchant consent; removing takes effect on approval. After changing `permissions` during development, restart `whatalo dev` so the development installation re-syncs `granted_scopes`.
Sources: [Scopes & Permissions](https://developers.whatalo.com/docs/plugin-sdk/configuration/scopes-and-permissions), [Updates & Versioning](https://developers.whatalo.com/docs/plugin-sdk/updates-and-versioning#scope-changes-and-merchant-consent).

## Source conflicts

- **Portal URL.** CLI Overview lists flag → `WHATALO_DEVELOPER_PORTAL_URL` → `http://localhost:3002` for every command; the inspected 1.6.0 CLI resolves per command as in [First run](#first-run). Check your installed CLI.
- **Node.js.** Docs and `whatalo` engines say 18+; the starter's Vite 7 requires `^20.19.0 || >=22.12.0`.

- **Scope list.** The plugin docs table lists the 15 scopes above. `@whatalo/protocol@1.4.0` (the type behind `AppPermission`) also defines `read:categories`, `write:categories`, and `read:checkout_drafts`. Plugin docs do not document those for plugins; do not request them without public guidance.
- **Categories.** The manifest page lists `payment` and `payments`; `@whatalo/protocol@1.4.0` contains `payments` but not `payment`. Use `payments`.
- **Data Bridge manifest example** uses `scopes: [...]` and `defineApp` from the package root; the manifest contract field is `permissions`, imported from `@whatalo/plugin-sdk/manifest` in the starter.
- **Webhooks overview manifest example** uses `pluginId`; the manifest field is `id`.
- **Build Your First Plugin** shows a generated manifest without `shortDescription`; the 1.6.0 starter template includes it and publication requires it.
- **Version format.** The manifest requires `\d+\.\d+\.\d+`; Updates & Versioning lists `1.0.0-beta.1` as allowed. Follow the manifest contract.
- **Counts.** The overview cites 15 scopes and 13 webhook events; Platform Overview and Event Reference cite 11 public events. This skill does not rely on these counts.
