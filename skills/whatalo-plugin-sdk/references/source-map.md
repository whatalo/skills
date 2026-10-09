# Source map

This skill summarizes the public sources below; it is not a complete copy of them. Sources were checked on 2026-10-09. Portal pages were read through their `.mdx` source (append `.mdx` to the page URL); the URLs listed here are the human-readable pages.

## Versions covered

From the [release history](https://developers.whatalo.com/docs/plugin-sdk/release-history) entries "Store-isolated development previews and explicit cleanup" (2026-10-09, `latest`) and "Stable developer toolchain with plugin review and theme authoring migrations" (2026-10-07), and the npm registry:

| Package | Inspected version | npm `latest` on 2026-10-09 |
| --- | --- | --- |
| `@whatalo/plugin-sdk` | 1.5.0 | 1.5.0 |
| `whatalo` (CLI wrapper of `@whatalo/cli`) | 1.7.0 (full inspection on 1.6.0) | 1.7.0 |
| `@whatalo/cli`, `@whatalo/cli-kit` | 1.7.0 (full inspection on 1.6.0) | 1.7.0 |
| `create-whatalo-plugin` (starter templates) | 1.7.0 (full inspection on 1.6.0) | 1.7.0 |
| `@whatalo/protocol` (scopes, events, categories) | 1.4.0 | dependency of SDK 1.5.0 and `@whatalo/cli` 1.7.0 |

The 1.7.0 release keeps SDK 1.5.0 and protocol 1.4.0. Diffing the published 1.6.0 and 1.7.0 tarballs showed that the starter templates differ only in `src/components/whatalo-ui/use-theme-sync.ts` and that `@whatalo/cli` adds the `dev clean` subcommand, adds `CLI_VERSION`/`cliVersion` reporting to `doctor`, and extracts shared dev API/store selectors; statements marked 1.6.0 elsewhere come from the full 1.6.0 inspection. For any other installed version, re-check the docs and package declarations.

## Discovery and fetch results

| Source | Method | Result |
| --- | --- | --- |
| `https://developers.whatalo.com/docs/plugin-sdk/llms.txt` | index | 61 page links |
| `https://developers.whatalo.com/docs/plugin-sdk` navigation | links in rendered HTML | 61 pages plus 1 extra link |
| `https://developers.whatalo.com/docs/api/llms.txt` | index (REST API, not the plugin SDK) | used only for context |
| Plugin SDK pages (`.mdx`) | fetched | 61 succeeded, 1 failed |
| `/docs/plugin-sdk/publishing/review-process` | linked from the SDK overview "Publishing" row | HTTP 404; the live page is [Review Process](https://developers.whatalo.com/docs/plugin-sdk/review-process) |
| `https://developers.whatalo.com/sitemap.xml` | sitemap | returns an HTML page, not a sitemap |
| Internal `/docs/plugin-sdk/...` links inside the 61 pages | HTTP check of 50 unique targets | 46 return 200; 4 return 404 (listed in [publishing-and-operations.md](publishing-and-operations.md#source-conflicts)) |

## Portal pages by reference

| Reference | Pages (all under `https://developers.whatalo.com/docs/plugin-sdk`) |
| --- | --- |
| [getting-started.md](getting-started.md) | [overview](https://developers.whatalo.com/docs/plugin-sdk), [quick-start](https://developers.whatalo.com/docs/plugin-sdk/quick-start), [getting-started/prerequisites](https://developers.whatalo.com/docs/plugin-sdk/getting-started/prerequisites), [getting-started/platform-overview](https://developers.whatalo.com/docs/plugin-sdk/getting-started/platform-overview), [getting-started/plugin-architecture](https://developers.whatalo.com/docs/plugin-sdk/getting-started/plugin-architecture), [getting-started/build-your-first-plugin](https://developers.whatalo.com/docs/plugin-sdk/getting-started/build-your-first-plugin), [configuration/plugin-manifest](https://developers.whatalo.com/docs/plugin-sdk/configuration/plugin-manifest), [configuration/project-config](https://developers.whatalo.com/docs/plugin-sdk/configuration/project-config), [configuration/environment-variables](https://developers.whatalo.com/docs/plugin-sdk/configuration/environment-variables), [configuration/scopes-and-permissions](https://developers.whatalo.com/docs/plugin-sdk/configuration/scopes-and-permissions) |
| [ui-contract.md](ui-contract.md) | [ui-components/overview](https://developers.whatalo.com/docs/plugin-sdk/ui-components/overview), [ui-components/layout](https://developers.whatalo.com/docs/plugin-sdk/ui-components/layout), [ui-components/content](https://developers.whatalo.com/docs/plugin-sdk/ui-components/content), [ui-components/actions](https://developers.whatalo.com/docs/plugin-sdk/ui-components/actions), [ui-components/hooks](https://developers.whatalo.com/docs/plugin-sdk/ui-components/hooks), [best-practices/design-guidelines](https://developers.whatalo.com/docs/plugin-sdk/best-practices/design-guidelines), [app-bridge/theme-integration](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/theme-integration), [app-bridge/ui-actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions), [best-practices/performance](https://developers.whatalo.com/docs/plugin-sdk/best-practices/performance) |
| [app-bridge.md](app-bridge.md) | [app-bridge/overview](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/overview), [app-bridge/context-and-session](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/context-and-session), [app-bridge/navigation](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/navigation), [app-bridge/ui-actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions), [api-client/data-bridge](https://developers.whatalo.com/docs/plugin-sdk/api-client/data-bridge), [api-client/authentication](https://developers.whatalo.com/docs/plugin-sdk/api-client/authentication), [api-client/session-tokens](https://developers.whatalo.com/docs/plugin-sdk/api-client/session-tokens), [connect-your-own-accounts](https://developers.whatalo.com/docs/plugin-sdk/connect-your-own-accounts), [best-practices/error-handling](https://developers.whatalo.com/docs/plugin-sdk/best-practices/error-handling), [best-practices/security](https://developers.whatalo.com/docs/plugin-sdk/best-practices/security) |
| [rest-api-client.md](rest-api-client.md) | [api-client/overview](https://developers.whatalo.com/docs/plugin-sdk/api-client/overview), [api-client/errors-and-rate-limits](https://developers.whatalo.com/docs/plugin-sdk/api-client/errors-and-rate-limits), [api-client/products](https://developers.whatalo.com/docs/plugin-sdk/api-client/products), [api-client/orders](https://developers.whatalo.com/docs/plugin-sdk/api-client/orders), [api-client/customers](https://developers.whatalo.com/docs/plugin-sdk/api-client/customers), [api-client/discounts](https://developers.whatalo.com/docs/plugin-sdk/api-client/discounts), [api-client/inventory](https://developers.whatalo.com/docs/plugin-sdk/api-client/inventory), [api-client/store](https://developers.whatalo.com/docs/plugin-sdk/api-client/store), [api-client/webhooks-api](https://developers.whatalo.com/docs/plugin-sdk/api-client/webhooks-api) |
| [webhooks.md](webhooks.md) | [webhooks/overview](https://developers.whatalo.com/docs/plugin-sdk/webhooks/overview), [webhooks/event-reference](https://developers.whatalo.com/docs/plugin-sdk/webhooks/event-reference), [webhooks/handling-webhooks](https://developers.whatalo.com/docs/plugin-sdk/webhooks/handling-webhooks), [webhooks/verification](https://developers.whatalo.com/docs/plugin-sdk/webhooks/verification), [cli-reference/webhook-trigger](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/webhook-trigger), [cli-reference/logs](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/logs) |
| [billing.md](billing.md) | [billing/overview](https://developers.whatalo.com/docs/plugin-sdk/billing/overview), [billing/plans-and-pricing](https://developers.whatalo.com/docs/plugin-sdk/billing/plans-and-pricing), [billing/subscription-flow](https://developers.whatalo.com/docs/plugin-sdk/billing/subscription-flow), [billing/billing-sdk-reference](https://developers.whatalo.com/docs/plugin-sdk/billing/billing-sdk-reference), [billing/revenue-and-payouts](https://developers.whatalo.com/docs/plugin-sdk/billing/revenue-and-payouts) |
| [publishing-and-operations.md](publishing-and-operations.md) | [cli-reference/overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview), [cli-reference/login](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/login), [cli-reference/init](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/init), [cli-reference/dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev), [cli-reference/env](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/env), [cli-reference/validate](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/validate), [cli-reference/deploy](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/deploy), [cli-reference/logs](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/logs), [cli-reference/utility-commands](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/utility-commands), [distribution](https://developers.whatalo.com/docs/plugin-sdk/distribution), [private-invitations](https://developers.whatalo.com/docs/plugin-sdk/private-invitations), [review-process](https://developers.whatalo.com/docs/plugin-sdk/review-process), [updates-and-versioning](https://developers.whatalo.com/docs/plugin-sdk/updates-and-versioning), [release-history](https://developers.whatalo.com/docs/plugin-sdk/release-history) |

## Package sources

Read from the published npm tarballs (no installation):

| Source | What was checked |
| --- | --- |
| [`create-whatalo-plugin@1.6.0`](https://www.npmjs.com/package/create-whatalo-plugin/v/1.6.0), `dist/templates/react-vite/` | Canonical starter: `src/components/whatalo-ui/*` (components, `styles.css`, hooks), `src/app.tsx`, `src/main.tsx`, `src/theme-bootstrap.ts`, `src/lib/plugin-api.ts`, `src/webhooks/verify.ts`, `server/*`, `vite.config.ts`, `whatalo.app.ts.hbs`, `whatalo.app.toml.hbs`, `package.json.hbs`, `_env.example`, `_gitignore` |
| [`@whatalo/plugin-sdk@1.5.0`](https://www.npmjs.com/package/@whatalo/plugin-sdk/v/1.5.0) | `package.json` exports, `README.md`, bridge/client/manifest/webhooks type declarations, `useAppBridge` implementation, `DATA_RESOURCE_SCOPE`, client retry code |
| [`@whatalo/protocol@1.4.0`](https://www.npmjs.com/package/@whatalo/protocol/v/1.4.0) | scope tokens, webhook event names, marketplace categories |
| [`whatalo@1.6.0`](https://www.npmjs.com/package/whatalo/v/1.6.0) and [`@whatalo/cli@1.6.0`](https://www.npmjs.com/package/@whatalo/cli/v/1.6.0) | command and option registrations |
| [`create-whatalo-plugin@1.7.0`](https://www.npmjs.com/package/create-whatalo-plugin/v/1.7.0) | template diff against 1.6.0; `use-theme-sync.ts` |
| [`whatalo@1.7.0`](https://www.npmjs.com/package/whatalo/v/1.7.0), [`@whatalo/cli@1.7.0`](https://www.npmjs.com/package/@whatalo/cli/v/1.7.0), [`@whatalo/cli-kit@1.7.0`](https://www.npmjs.com/package/@whatalo/cli-kit/v/1.7.0) | dependency pins; command registration diff against 1.6.0; `dev clean` options, statuses, and JSON receipt keys |

## Development workflow sources

[development-workflow.md](development-workflow.md) was authored against the maintainers' implementation of the development-preview workflow and re-checked against the released CLI family 1.7.0 and the public pages below. The published `@whatalo/cli@1.7.0` registers `dev clean` with `--store`/`-s`, `--reset`, `--portal-url`, and `--json`, uses the statuses `restored`, `removed`, and `noop`, and prints the JSON receipt keys `status`, `storePublicId`, `storeName`, `plugin`.

| Official page | HTTP status on 2026-10-09 (after the 1.7.0 release) |
| --- | --- |
| [whatalo dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev) | 200; describes store isolation, preview retention on exit, and `dev clean` |
| [whatalo dev clean](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev-clean) | 200 |
| [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview) | 200; lists `whatalo dev clean` |
| [Review Process](https://developers.whatalo.com/docs/plugin-sdk/review-process) | 200 |
| [Release history](https://developers.whatalo.com/docs/plugin-sdk/release-history) | 200; "Store-isolated development previews and explicit cleanup" entry |

## Example repository

[bellopushon/whatalo-plugin-examples](https://github.com/bellopushon/whatalo-plugin-examples) at commit `a913130` (2026-10-03). Its README states it is "an independent example project, **not an official Whatalo-endorsed app**". The Quick Start links to it as a complete example. This skill uses it only to illustrate patterns that the portal or packages also document; its extra UI (`skeleton.tsx`, `toolkit-*` classes) is not part of the canonical starter and is not prescribed.
