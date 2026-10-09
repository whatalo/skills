---
name: whatalo-plugin-sdk
description: "Trigger: Whatalo plugin, @whatalo/plugin-sdk, whatalo CLI, App Bridge, whatalo-ui, plugin webhooks or billing. Build and review Whatalo admin plugins from public docs."
license: Apache-2.0
metadata:
  version: "0.1.0"
  inspected-plugin-sdk: "1.5.0"
  inspected-create-whatalo-plugin: "1.6.0"
  inspected-on: "2026-10-09"
---

## Activation Contract

Load for work on a Whatalo admin plugin: scaffolding, manifest/TOML, App Bridge, Data Bridge, session tokens, `WhataloClient`, webhooks, billing, account connection, dev preview, deploy, review, or UI. Not for themes or direct REST/OAuth integrations.

## Hard Rules

- This skill routes and constrains; it does not replace the official docs. Read the linked portal pages for each contract before implementing ([source-map.md](references/source-map.md)).
- Compare installed `@whatalo/plugin-sdk`, `whatalo`, and starter versions with the inspected ones; on mismatch re-check docs, `--help`, and type declarations.
- If docs, package, starter, or CLI help differ, or behavior is undocumented, say so and stop. Never invent APIs, values, events, flags, or counts.
- `appId` is always the manifest `id` (`app.id`), never `WHATALO_CLIENT_ID`.
- Secrets and `WhataloClient` stay on the server.
- Verify webhook HMAC over the raw body with the right per-installation secret; deduplicate on `delivery_id`.
- No vendor credential inputs or login gate in the iframe; use `authUrl` + `bridge.openExternal()`.
- Before any `whatalo dev clean` instruction, run `whatalo dev clean --help` and proceed only if it matches [development-workflow.md](references/development-workflow.md).

### Mandatory UI contract (maintainer approval policy)

Stricter than the public docs; maintainer policy, not platform automation. Details: [ui-contract.md](references/ui-contract.md).

1. Canonical starter components from `src/components/whatalo-ui` with their real props; semantic HTML inside them is allowed for accessibility; no alternate design systems.
2. Only existing `--wui-*` tokens and `.wui-*` classes; no new tokens, hex colors, or fonts.
3. Every page is `Page` + `PageHeader`, centered by `Page`, and works in narrow iframes.
4. Both themes work via `useThemeSync()` and the starter theme bootstrap.
5. Host-managed sizing: `html, body { overflow: hidden; background: transparent; }` + `useAutoResize()`; no inner scroll container, no double scroll.
6. Loading, empty, and error states; keep starter accessibility attributes.

## Decision Gates

| Need | Use | Reference |
| --- | --- | --- |
| Shell context, current page | `useWhataloContext()` | [app-bridge.md](references/app-bridge.md#hooks-whataloplugin-sdkbridge) |
| Read-only store data in the iframe | `useWhataloData()` | [app-bridge.md](references/app-bridge.md#data-bridge-usewhatalodata) |
| Writes, exports, trusted logic | backend + `WhataloClient` | [rest-api-client.md](references/rest-api-client.md) |
| Frontend → own backend auth | `sessionToken()` + `verifyWhataloSessionToken` | [app-bridge.md](references/app-bridge.md#session-tokens-and-backend-auth) |
| Store events | manifest `webhooks` + SDK handler | [webhooks.md](references/webhooks.md) |
| Paid plans (UX gating only) | `bridge.billing.*` | [billing.md](references/billing.md) |
| Preview, stop, clean a dev store | `whatalo dev`, `whatalo dev clean` | [development-workflow.md](references/development-workflow.md) |
| Validate, deploy, review | `whatalo validate`, `whatalo deploy` | [publishing-and-operations.md](references/publishing-and-operations.md) |

## Execution Steps

1. Read [getting-started.md](references/getting-started.md) and the official pages it links.
2. Load only the references the task needs, then their linked docs.
3. Apply the UI contract to every UI change.
4. Validate with `whatalo validate` and `pnpm type-check`; preview with `whatalo dev`.

## Output Contract

Return changed files, scopes and events used, the docs/reference sections behind each SDK or CLI call, how each UI rule is met, and any undocumented or version-mismatched behavior you refused to assume.

## References

- [source-map.md](references/source-map.md) — inspected sources, release-dependent sources
- [getting-started.md](references/getting-started.md) — setup, manifest, TOML, identity, env, scopes
- [ui-contract.md](references/ui-contract.md) — components, tokens, layout, theme, resize
- [app-bridge.md](references/app-bridge.md) — bridge, Data Bridge, session tokens, account connection
- [rest-api-client.md](references/rest-api-client.md) — `WhataloClient`
- [webhooks.md](references/webhooks.md) — events, verification, secrets
- [billing.md](references/billing.md) — plans and `bridge.billing`
- [development-workflow.md](references/development-workflow.md) — dev preview scope, stop, clean
- [publishing-and-operations.md](references/publishing-and-operations.md) — validate, deploy, review, troubleshooting
