# Webhooks, connection lifecycle, uninstall

Inspected against `@whatalo/plugin-sdk` 1.5.0 and `whatalo` 1.7.0 (audited baseline and delta: see source map); summary only, read the linked pages for full contracts. Sources: [source-map.md](source-map.md).

## Subscribing

Declare events in the manifest and set `webhookUrl` (all events go there). Only declare events you handle; `description` is shown on the install permission screen. Webhooks are optional for publication, but declared ones need a valid `webhookUrl`. `client.webhooks.*` registers subscriptions at runtime instead (see [rest-api-client.md](rest-api-client.md)).

```ts
webhookUrl: "https://my-plugin.example.com/webhooks/whatalo",
webhooks: [{ event: "order.created", description: "Track new orders" }],
```

During `whatalo dev`, `webhookUrl` is set to `{tunnelUrl}/webhooks/whatalo` automatically.
Sources: [Webhooks Overview](https://developers.whatalo.com/docs/plugin-sdk/webhooks/overview), [whatalo dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev#dev-session-details).

## Public events

| Group | Events |
| --- | --- |
| Orders | `order.created`, `order.updated`, `order.cancelled`, `order.completed` |
| Products | `product.created`, `product.updated`, `product.deleted` |
| Customers | `customer.created`, `customer.updated` |
| Checkout | `checkout.completed`, `checkout.abandoned` |

- Each order status transition emits one event: `completed` → `order.completed`; `cancelled` or `returned` → `order.cancelled`; any other status → `order.updated`. Payment status changes emit `order.updated` with `order.payment_status` and no `order.status`.
- `checkout.abandoned`: `event_id` equals `checkout.id`, `customer` has no `id`, and `checkout.recovery_url` links to the cart.
- Treat unknown payload fields as additive.

This list matches `@whatalo/protocol@1.4.0`. Source: [Event Reference](https://developers.whatalo.com/docs/plugin-sdk/webhooks/event-reference).

## Delivery

- `POST`, `Content-Type: application/json`, `User-Agent: Whatalo-Webhooks/1.0`.
- Headers: `X-Webhook-Id` (delivery ID, stable across retries), `X-Webhook-Event`, `X-Webhook-Signature`, `X-Webhook-Timestamp` (Unix seconds).
- The event name is **only** in `X-Webhook-Event`, not in the body.
- Body: `delivery_id` (= `X-Webhook-Id`), `event_id` (affected entity's public ID), `occurred_at`, the entity object, and `store { id, name?, timezone }`.
- Respond with any `2xx` within 15 s. `3xx` and `4xx` are failures without retry; `5xx`, network errors, and timeouts are retried for up to 3 total attempts (waits of 1 s, then 2 s).

## Verification

Signature = hex HMAC-SHA256 over `` `${timestamp}.${rawBody}` ``. Verify before parsing JSON, compare in constant time, reject timestamps more than 300 s in the past or future, and only process declared event types.

Options, all requiring the **raw** body:

- SDK adapters: `createWebhookHandler({ secret, handlers, onUnhandledEvent? })` from `@whatalo/plugin-sdk/adapters/nextjs`, `/hono`, or `/express`. With Express, mount `express.raw({ type: "application/json" })` on the route, before any global `express.json()`. A thrown handler returns `500` (triggers retry); returning normally returns `200`.
- `verifyWebhook({ payload, signature, timestamp, secret })` from `@whatalo/plugin-sdk/webhooks` (default 300 s window).
- The starter's `src/webhooks/verify.ts` `verifyWhataloWebhook(headers, rawBody, secret)`, used by `POST /webhooks/whatalo` in `server/index.ts`. The starter returns `501` until `WHATALO_WEBHOOK_SECRET` is set.

Handler context: `context.deliveryId`, `context.event`, `context.timestamp`.
Sources: [Handling Webhooks](https://developers.whatalo.com/docs/plugin-sdk/webhooks/handling-webhooks), [Verification & Security](https://developers.whatalo.com/docs/plugin-sdk/webhooks/verification), starter `server/index.ts`.

## Secrets

Three distinct secrets; never treat one as the others:

- **Installation signing secret.** Marketplace app deliveries are signed with a per-installation, 64-character hex secret, revealed once at install time with the raw API key, independent from API keys, and replaced on reinstall. If lost, the merchant must reinstall.
- **Custom subscription secret.** `client.webhooks.create({ url, events, secret })` takes a secret you choose for that API-registered subscription.
- **Local test secret.** `whatalo webhook trigger` signs with `--secret` or `WHATALO_WEBHOOK_SECRET` (16–128 chars); configure your local handler with the same value.

`createWebhookHandler`, `verifyWebhook`, and the starter verifier each take one `secret: string`. A single global secret does not validate every installation. In production, select the stored secret for the delivering installation, verify the HMAC with it, and only then trust the payload. Before verification, values such as `store.id` in the body are lookup hints only, not identity.

Public docs gap: they do not document how a backend receives and stores each installation's secret (beyond "revealed once at install time"), nor which store identifier to key it by. Do not invent a provisioning API.
Sources: [Verification & Security](https://developers.whatalo.com/docs/plugin-sdk/webhooks/verification#where-the-secret-comes-from), [whatalo webhook trigger](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/webhook-trigger).

## Idempotency and ordering

Store processed `delivery_id` values and skip duplicates. Never deduplicate on `event_id`: successive events for the same entity share it. Order events by `occurred_at`.

## Testing and logs

```sh
whatalo webhook trigger --list --secret $WHATALO_WEBHOOK_SECRET
whatalo webhook trigger order.created --secret $WHATALO_WEBHOOK_SECRET
whatalo logs --status failed --follow
```

Triggers work only against development stores (production stores return `403`), target the manifest `webhookUrl` or the active tunnel, and accept `--payload <file>` that must match the event's runtime shape (else `400`). `whatalo logs` supports `--event`, `--status delivered|failed`, `--store`, `--limit` (max 100), `--detail <id>`, `--json`.
Sources: [whatalo webhook trigger](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/webhook-trigger), [whatalo logs](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/logs).

## Connection and uninstall lifecycle

- Account connection to the vendor's own service: see [app-bridge.md](app-bridge.md#vendor-account-connection).
- Uninstall: the public docs define **no** uninstall webhook event and no uninstall callback for plugins. Do not subscribe to or branch on an uninstall event. The docs require deleting stored merchant data "when your plugin is removed or when a merchant requests deletion" but do not document how a plugin learns of removal. Report this gap instead of inventing a mechanism.

Sources: [Security Best Practices](https://developers.whatalo.com/docs/plugin-sdk/best-practices/security#7-do-not-store-unnecessary-merchant-data), [Customers](https://developers.whatalo.com/docs/plugin-sdk/api-client/customers#data-privacy-considerations).

## Source conflicts

- Env var names differ by page: `WHATALO_WEBHOOK_SIGNING_SECRET` (adapters, verification), `WHATALO_WEBHOOK_SECRET` (starter, CLI trigger), and Authentication says `WHATALO_CLIENT_SECRET` verifies webhooks. The Verification page defines the signing secret as a separate per-installation value; do not use the client secret.
- Security Best Practices reads `x-whatalo-hmac-sha256` and omits the timestamp; the delivery contract uses `X-Webhook-Signature` plus `X-Webhook-Timestamp`.
- Webhooks Overview says logs do not capture the endpoint response body; `whatalo logs --detail` lists a "Response" field.
- The independent example branches on `payload.event` and an `app.uninstalled` event; neither is in the public contract.
- The public docs do not explain how a production backend receives each installation's API key and webhook signing secret beyond "revealed once at install time".
