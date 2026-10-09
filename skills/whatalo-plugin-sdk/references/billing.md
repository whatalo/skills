# Billing

Inspected against `@whatalo/plugin-sdk` 1.5.0; summary only, read the linked pages for full contracts. Sources: [source-map.md](source-map.md).

## Setup

1. Set `pricing: "paid"` in the manifest.
2. Create plans in the Developer Portal (not in code). Up to 5 active plans per plugin; deactivate instead of deleting.
3. Use `bridge.billing.*` from the iframe.

Plan fields: `name`, `slug` (unique, immutable once a plan has active subscribers), `description?`, `price` (major units), `currency` (USD only), `interval` (`monthly` | `annual`, recurring plans), `trialDays` (0–90, default 0), `features`, `isPopular`, `sortOrder`. Pricing types: `recurring` and `one_time` available; `usage` documented as "Coming soon".
Sources: [Billing Overview](https://developers.whatalo.com/docs/plugin-sdk/billing/overview), [Plans & Pricing](https://developers.whatalo.com/docs/plugin-sdk/billing/plans-and-pricing).

## Bridge API

Methods: `getPlans`, `getSubscription` (or `null`), `requestSubscription(planSlug)` (host redirects to approval; may not return), `cancelSubscription` (at period end), `reactivateSubscription`, `switchPlan(newPlanSlug)` (prorated). Types and failure reasons: [Billing SDK Reference](https://developers.whatalo.com/docs/plugin-sdk/billing/billing-sdk-reference).

`BillingSubscriptionResponse`: `planSlug`, `planName`, `status`, `trialEndsAt`, `currentPeriodEnd`, `cancelAtPeriodEnd`, `canceledAt`. Status values: `pending`, `trialing`, `active`, `past_due`, `canceled`, `expired`.

Billing methods **throw** on failure (they do not return `{ success: false }`); wrap them in try/catch and show a toast with `variant: "error"`. Billing actions share the bridge limit of 10 per 10 s and the 5 s action timeout.
Sources: [Billing SDK Reference](https://developers.whatalo.com/docs/plugin-sdk/billing/billing-sdk-reference), [Subscription Flow](https://developers.whatalo.com/docs/plugin-sdk/billing/subscription-flow).

## Gating features

Treat `active` and `trialing` as entitled. Fetch the subscription after `isReady`, render loading and error states, and show an upgrade prompt otherwise (pattern in [Subscription Flow](https://developers.whatalo.com/docs/plugin-sdk/billing/subscription-flow#gating-features-by-subscription)). This is UX gating only, not authorization: anything the iframe decides can be bypassed. The public docs document no server-side billing or entitlement endpoint; do not invent one, and report the gap if a backend must enforce entitlements.

## Platform rules

- One subscription per installation; trials once per installation (reinstalling does not re-grant). Cancelling during a trial ends at trial expiry with no charge.
- Charges are line items on the merchant's platform subscription.
- Default commission 5%, snapshotted per charge. Payouts need ≥ $50.00 USD available, a verified payout method, and no other pending request.

Sources: [Billing Overview](https://developers.whatalo.com/docs/plugin-sdk/billing/overview), [Revenue & Payouts](https://developers.whatalo.com/docs/plugin-sdk/billing/revenue-and-payouts).
