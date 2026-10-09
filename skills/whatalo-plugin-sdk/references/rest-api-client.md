# REST operations with `WhataloClient`

Inspected against `@whatalo/plugin-sdk` 1.5.0. Server-side only; summary only, read the linked pages for full contracts. Sources: [source-map.md](source-map.md).

## Client

```ts
import { WhataloClient } from "@whatalo/plugin-sdk";

const client = new WhataloClient({ apiKey: process.env.WHATALO_API_KEY!, retries: 2 });
```

| Option | Default | Notes |
| --- | --- | --- |
| `apiKey` | required | Store-scoped `wk_live_*` / `wk_test_*`; sent as `X-API-Key` |
| `baseUrl` | `https://api.whatalo.com/v1` | |
| `timeout` | `30000` ms | |
| `retries` | `0` | 0–3, 5xx only, exponential backoff |
| `fetch`, `onRequest`, `onResponse` | — | testing and logging hooks |

`client.rateLimit` exposes `{ limit, remaining, reset }` from `X-RateLimit-*` headers after each call. Never ship the key to the browser, including through build-time env inlining. In development, `whatalo dev` writes a test key to `.env`.
Sources: [API Client Overview](https://developers.whatalo.com/docs/plugin-sdk/api-client/overview), [Authentication](https://developers.whatalo.com/docs/plugin-sdk/api-client/authentication).

## Return shapes

In SDK 1.5.0 every method resolves to `{ data }`; `list` resolves to `{ data, pagination: { page, per_page, total, total_pages } }`. Read fields from `.data` (for example `(await client.store.get()).data.currency`).
Source: SDK 1.5.0 `client/index.d.ts`.

## Operations

Methods, HTTP mapping, and required scope per method are on each resource page: [products](https://developers.whatalo.com/docs/plugin-sdk/api-client/products), [orders](https://developers.whatalo.com/docs/plugin-sdk/api-client/orders), [customers](https://developers.whatalo.com/docs/plugin-sdk/api-client/customers), [discounts](https://developers.whatalo.com/docs/plugin-sdk/api-client/discounts), [inventory](https://developers.whatalo.com/docs/plugin-sdk/api-client/inventory), [store](https://developers.whatalo.com/docs/plugin-sdk/api-client/store), [webhooks](https://developers.whatalo.com/docs/plugin-sdk/api-client/webhooks-api). Reads need `read:*`, mutations need `write:*`; orders allow only `updateStatus` and `updatePaymentStatus`. `categories` has no SDK page; confirm with installed types before use.

Values:

- Order `status`: `pending`, `confirmed`, `in_progress`, `completed`, `cancelled`, `returned`. Only valid transitions are accepted (`ValidationError` otherwise).
- `payment_status`: `pending`, `paid`, `refunded`, `failed`.
- Product `product_status`: `active`, `draft`, `archived`.
- Discount `type`: `percentage` or `fixed`.
- Inventory `quantity` is a signed delta; `reason` is a non-empty free-text audit string.

Use pagination; never fetch all records in one request. Sources: the resource pages in [source-map.md](source-map.md#portal-pages-by-reference).

## Errors

All errors extend `WhataloAPIError` (`statusCode`, `code`, `requestId`, `message`):

| Class | Status | Extra |
| --- | --- | --- |
| `AuthenticationError` | 401 | — |
| `AuthorizationError` | 403 | `requiredScope` |
| `NotFoundError` | 404 | `resourceType`, `resourceId` |
| `ValidationError` | 422 | `fieldErrors: { field, message }[]` |
| `RateLimitError` | 429 | `retryAfter` (seconds) |
| `InternalError` | 500 | — |

- 5xx retries wait 1 s, 2 s, 4 s (attempts 1–3) in SDK 1.5.0 and the Errors page.
- 429 is never retried automatically; schedule a retry from `retryAfter`, and slow down batches when `client.rateLimit.remaining` is low.
- Log `requestId` for support. Re-throw unknown errors.

Source: [Errors & Rate Limits](https://developers.whatalo.com/docs/plugin-sdk/api-client/errors-and-rate-limits).

## Source conflicts

- Several resource pages read fields directly from the result (`product.name`, `store.currency`, `discount.code`) or use `pagination.pages`; SDK 1.5.0 types return `{ data }` and `pagination.total_pages`.
- Overview says 5xx backoff is 2 s, 4 s, 8 s; the Errors page and SDK code use 1 s, 2 s, 4 s.
- Overview says 429 is not retried "unless `retries > 0`"; SDK 1.5.0 throws `RateLimitError` on 429 regardless of `retries`.
- `products.count(status?)` is typed `"active" | "inactive" | "all"` in SDK 1.5.0; the docs example passes `"archived"`.
- `discounts.create` requires `name` in the SDK 1.5.0 type; the docs examples omit it.
- The Data Bridge comparison table cites "100 req / 10 s" for the API; the rate-limit pages show the limit only as a header value. Read `client.rateLimit` instead of hard-coding a limit.
