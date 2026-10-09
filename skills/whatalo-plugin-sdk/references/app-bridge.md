# App Bridge, Data Bridge, actions, session tokens, backend auth

Inspected against `@whatalo/plugin-sdk` 1.5.0; summary only, read the linked pages for full contracts. Sources: [source-map.md](source-map.md).

## Protocol and security

- Messages: `whatalo:init` (host → plugin, parent origin), `whatalo:action` (plugin → host), `whatalo:context` (host → plugin), `whatalo:ack` (host → plugin).
- Handshake: init → plugin sends `ready` → host sends context → `isReady === true`. Actions sent before the handshake are queued.
- The plugin posts only to the origin received in `whatalo:init`; the host drops messages whose origin is not the registered `appUrl`; an `appUrl` on the admin origin is refused.
- Every action resolves to `ActionAck { type, actionId, success, error? }`. Timeout is 5 s (`error: "timeout"`). Other errors: `"rate_limited"`, `"invalid_url"`, `"invalid_path"`, `"out_of_bounds"`.
- Per-plugin sliding 10 s limits, shared across tabs: toast 3, navigate 5, modal 3, resize 30, billing 10.

Sources: [App Bridge Overview](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/overview), [Plugin Architecture](https://developers.whatalo.com/docs/plugin-sdk/getting-started/plugin-architecture), [UI Actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions#actionack-type).

## Hooks (`@whatalo/plugin-sdk/bridge`)

| Hook | Returns |
| --- | --- |
| `useWhataloContext()` | `storeId`, `storeName`, `user { id, name, email, role }`, `appId`, `currentPage`, `locale`, `theme`, `initialHeight`, optional `orderId`, `orderStatus`, `productId`, plus `isReady` |
| `useWhataloAction()` | `sendAction`, `showToast`, `navigate`, `openExternal`, `openModal`, `closeModal`, `resize`, `billing` |
| `useAppBridge()` | `storeId`, `storeName`, `locale`, `theme`, `user`, `appId`, `isReady`, `toast.show`, `navigate`, `openExternal`, `modal.open/close`, `resize`, `billing` — **no `currentPage`** |
| `useWhataloData()` | `products`, `orders`, `customers`, `categories`, `store` (see Data Bridge) |

Rules:

- Gate logic on `isReady`. Before it, defaults are empty strings, `locale "es"`, `theme "light"`, `initialHeight 200`, `user.role "owner"`.
- `storeId` is the public store ID, not a database UUID.
- `user.name` is a display label only; use `user.email` and `user.role` for identity-sensitive UI. Roles: `owner`, `admin`, `editor`, `staff`, `viewer`.
- `useWhataloContext()` is not a store metadata API; read timezone, currency, or domain via `useWhataloData().store.get()`.

Sources: [Context & Session](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/context-and-session), SDK 1.5.0 `bridge/index.d.ts` and `useAppBridge` implementation.

## Page routing

No client-side router. The host sends `currentPage` (the manifest page `path`) and re-sends context on sidebar navigation:

```tsx
const context = useWhataloContext();
if (!context.isReady) return null;
switch (context.currentPage) {
  case "settings":
    return <SettingsPage />;
  default:
    return <DashboardPage />;
}
```

Keep each page self-contained. Source: starter `src/app.tsx`; [Navigation](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/navigation).

## Actions

- Toast: `bridge.toast.show(title, { description?, variant?: "success" | "error" | "warning" | "info" })`.
- Admin navigation: `bridge.navigate(path)`; only paths starting with `/store/` or `/admin/` are accepted.
- Modal: `bridge.modal.open({ title?, url, width?, height? })`, `bridge.modal.close()`. Per [UI Actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions#modal-security-rules), only URLs from the plugin's registered `appUrl` domain are allowed (localhost only in development mode); `javascript:`, `data:`, `blob:`, `file:`, `vbscript:` are blocked. Modal iframes have no `allow-same-origin`, and the bridge may not be ready at render time there; theme can arrive through the URL (see [ui-contract.md](ui-contract.md#theme)).
- Resize: `bridge.resize(height)`; prefer `useAutoResize()`.

Sources: [UI Actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions), [Navigation](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/navigation).

## Data Bridge (`useWhataloData()`)

Read-only store data from the iframe, proxied by the admin host; no backend or API key. The host checks the installation's `granted_scopes`.

| Resource | Scope | Methods |
| --- | --- | --- |
| `products` | `read:products` | `list({ page, per_page, status, search })`, `get(id)` |
| `orders` | `read:orders` | `list({ page, per_page, status })`, `get(id)` |
| `customers` | `read:customers` | `list({ page, per_page, search })`, `get(id)` |
| `categories` | `read:products` | `list({ page, per_page, search })`, `get(slug)` |
| `store` | `read:store` | `get()` |

- Responses: `{ data }` for `get`, `{ data, pagination: { page, per_page, total, total_pages } }` for `list`. Default page size 20, max 100 (larger values are clamped), pages below 1 become 1.
- Limit: 20 requests per 10 s across all resources; on excess the promise rejects with `"Rate limit exceeded"`.
- Calls reject (they do not return an ack); wrap in try/catch. Documented messages: `Missing scope: read:*`, `Plugin not installed`, `Rate limit exceeded`, `Resource ID required for get operation`, `* not found`, `Unauthorized`.
- Format money with the returned `currency`; never hard-code a display currency.
- Restart `whatalo dev` after changing `permissions`.

Source: [Data Bridge](https://developers.whatalo.com/docs/plugin-sdk/api-client/data-bridge).

## Session tokens and backend auth

Two credentials, never mixed:

| Layer | Credential | Use |
| --- | --- | --- |
| Iframe → your backend | Session token (JWT, 5 min) from `sessionToken()` | `Authorization: Bearer <token>` to **your** backend only |
| Your backend → Whatalo API | Store-scoped API key (`wk_live_*`/`wk_test_*`) | `WhataloClient`, server only |

Frontend:

- `sessionToken()` returns `{ token, expiresAt }`, caches the token, refreshes it when ≤60 s remain, and shares concurrent requests. Call it before each backend request; do not store tokens.
- The starter's `pluginFetch(path, options)` adds the bearer token and JSON headers, and on a `401` calls `clearSessionTokenCache()` and retries once.
- Session tokens are rejected by the Whatalo API; never send them there.

Backend (`@whatalo/plugin-sdk/server`):

```ts
import { verifyWhataloSessionToken } from "@whatalo/plugin-sdk/server";
import app from "../whatalo.app.js";

const claims = verifyWhataloSessionToken(token, env.WHATALO_CLIENT_SECRET, {
  expectedAppId: app.id,
});
```

- It throws on bad signature, expiry, malformed token, audience/app mismatch, non-numeric `iat`, or `iat` more than 30 s in the future. Return `401` on throw.
- Claims: `iss` (`"whatalo"`), `aud`, `sub`, `exp`, `iat`, `jti`, `storeId`, `appId`, `scopes`, `installationId`.
- Check `claims.scopes` for endpoint-specific permissions (`403` if missing). Record `jti` for operations that must not replay.
- Scope all persisted data by `claims.storeId`; do not trust a `storeId` sent by the browser.
- Keep the iframe and backend on one origin in development (starter Vite proxy); use an absolute backend URL if you deploy them on different origins.

Sources: [Authentication](https://developers.whatalo.com/docs/plugin-sdk/api-client/authentication), [Session Tokens](https://developers.whatalo.com/docs/plugin-sdk/api-client/session-tokens), starter `server/auth.ts` and `src/lib/plugin-api.ts`.

## Vendor account connection

For plugins that link an account on the vendor's own service:

1. Add `authUrl: "https://vendor.example.com/login"` (HTTPS, vendor origin, not the platform origin) to the manifest.
2. From a merchant-initiated Connect button, call `bridge.openExternal(authUrl)`. The host checks exact origin match (scheme, host, port), allows 3 attempts per 10 s, needs an active or development installation, and performs top-level navigation.
3. Your page receives `state` and `callbackUrl` query parameters. On the server, call `verifyConnectionState(state, WHATALO_CLIENT_SECRET)`; it returns `null` for invalid state, so reject `null`, and reject unless `claims.appId === app.id` and `claims.callbackUrl` equals the supplied `callbackUrl`. State expires after 180 s and is single-use. Do not log `state`.
4. Authenticate the account on your own top-level page. Derive `accountRef` from the account your backend authenticated, never from a browser-supplied value. Upsert your link keyed by `claims.storeId` + `claims.installationId`, then `buildConnectionCallbackUrl(claims.callbackUrl, { state, accountRef, issuedAt }, secret)` and redirect with HTTP `303` to that URL only; never substitute a merchant-supplied return URL. `issuedAt` is Unix seconds; `accountRef` is opaque, ≤256 chars, no credentials.

Rules: the iframe always loads without an auth gate; no credential inputs inside it (a review rejection criterion, also for private plugins); no frame-busting; protect your login page with `Content-Security-Policy: frame-ancestors 'none'`; the manifest `callbackUrl` field is not used. `{ allowLoopback: true }` is for explicit local development only.
Source: [Connect your own accounts](https://developers.whatalo.com/docs/plugin-sdk/connect-your-own-accounts).

## Source conflicts

- Context & Session, Navigation, Hooks, and Error Handling examples destructure `currentPage` from `useAppBridge()`; SDK 1.5.0 does not return it. Use `useWhataloContext()`.
- Theme Integration says modal theme arrives as `?theme=`; the starter reads `whatalo_theme`.
- Sandbox: Plugin Architecture lists `allow-scripts allow-forms allow-same-origin`; Connect your own accounts lists `allow-scripts allow-forms allow-same-origin allow-downloads`.
- SDK 1.5.0 also exposes a `geo` Data Bridge resource (scope `read:store`) that the Data Bridge page does not document; do not rely on it without public docs.
