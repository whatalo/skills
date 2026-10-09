# Development, testing, troubleshooting, deploy, review

Inspected against `whatalo` 1.6.0; summary only, read the linked pages for full contracts. Portal URL resolution differs between docs and the inspected CLI; see [getting-started.md](getting-started.md#first-run). Sources: [source-map.md](source-map.md).

## CLI commands

Full command and flag reference: [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview). Most used: `login`, `init`, `dev`, `env pull`, `validate`, `deploy`, `logs`, `webhook trigger`, `doctor`.

The installed 1.6.0 CLI also registers `whatalo changelog` (referenced from Updates & Versioning). Source: [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview).

## Local development (`whatalo dev`)

Sequence: read `whatalo.app.toml` → authenticate (auto-login) → load `whatalo.app.ts` → select dev store (`--store`, cache, sole available store, prompt) → run `build.dev_command` → start tunnel → register dev session → print console (`p` preview, `r` restart, `q` quit).

- The dev installation has `status: "development"`, is pinned to the sidebar, and copies manifest `permissions` into `granted_scopes`. `appUrl` becomes the tunnel URL and `webhookUrl` becomes `{tunnelUrl}/webhooks/whatalo`.
- Manifest edits hot-reload pages; permission changes need a restart of `whatalo dev`.
- Sessions last up to 8 hours (72 hours absolute); heartbeat every 30 minutes.
- Preview scope, stopping, and `whatalo dev clean`: see [development-workflow.md](development-workflow.md); run its compatibility check first.
- Options: `--store`/`-s`, `--tunnel-url <https-url>`, `--no-tunnel` (admin must be local), `--port`, `--reset` (ignores the cached store), `--verbose`.
- `pnpm dev` alone does not create a store session, test API key, or tunnel; open the plugin through the admin preview so it gets a session token.

Source: [whatalo dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev).

## Checks before deploy

```sh
pnpm type-check
whatalo validate --build
```

`whatalo validate` needs no auth and makes no API calls. It checks config, manifest, required dependencies (`@whatalo/plugin-sdk`, `react`, `vite`), hardcoded secrets, `.gitignore` (`.env`, `.whatalo/`, `node_modules/`; `--fix` repairs it), TypeScript, optional build, and port. `--strict` turns warnings into errors; `--json` for CI. Exit codes: 0 pass, 1 errors, 2 fatal.
Source: [whatalo validate](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/validate).

## Deploy

```sh
whatalo deploy --message "Add inventory sync"
whatalo deploy --force --message "Add inventory sync"
```

- `whatalo.app.ts` is required (also with `--no-build`); invalid manifests stop before any API call.
- No auto-login: run `whatalo login` first. `--force` is required in non-interactive CI.
- Version comes from `package.json` or `--set-version`; it must be `major.minor.patch` and not lower than the deployed version.
- Build timeout 120 s; the CLI checks `output_dir`.
- Receipt shows app URL, page paths, permissions, events, and `live` or `pending_review`.

| Current status | Deploy outcome |
| --- | --- |
| `draft` | Manifest updated, stays `draft` |
| `approved` (private) | Manifest updated, stays `approved` |
| `approved` (public) | Changes staged for review; live version unchanged until approval |
| `pending_review` | Rejected; nothing changes |
| `rejected` | Corrections saved; resubmit in the portal |

Source: [whatalo deploy](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/deploy).

## Distribution and review

- **Private** plugins are auto-approved and installable by the owner (`owner_only`) or, in `invitation` mode, by email-bound, single-use invitation links that expire after 7 days. Invitees need a paid-plan store. Source: [Distribution](https://developers.whatalo.com/docs/plugin-sdk/distribution), [Private Invitations](https://developers.whatalo.com/docs/plugin-sdk/private-invitations).
- **Public** plugins: after deploy, open **Saved review candidate** in the portal, then **Submit for Review** (or **Publish to Marketplace** for a private plugin).
- Submission gate: name 3–50 chars, `shortDescription` 10–160, `description` ≥100, valid category, at least one registered scope, valid HTTPS `appUrl`, at least one valid `adminUI.pages` entry, valid public events and `webhookUrl` if webhooks are declared.
- Reviewers check configuration, functionality (manual), scope usage, security, UX, policy, and reject credential inputs inside the iframe. Decisions arrive by email; expect a few business days.
- Pending manifest patches: an absent key keeps the live value, `null` removes the field, any other value replaces the whole field (no deep merge).
- Scope additions in updates require merchant consent; removals apply on approval.

Sources: [Review Process](https://developers.whatalo.com/docs/plugin-sdk/review-process), [Plugin Manifest](https://developers.whatalo.com/docs/plugin-sdk/configuration/plugin-manifest#pending-manifest-patch-semantics), [Updates & Versioning](https://developers.whatalo.com/docs/plugin-sdk/updates-and-versioning).

## Troubleshooting

| Symptom | Check |
| --- | --- |
| CLI talks to the wrong portal | `login`/`dev`: check `--portal-url`, `[dev] portal_url`, `WHATALO_DEVELOPER_PORTAL_URL`; other commands use the session portal, so `whatalo login --force` against the right portal (see [getting-started.md](getting-started.md#first-run)) |
| `No whatalo.app.toml found` | Run inside the project, or `whatalo init` first |
| `Plugin "{slug}" was not found in your account` | `plugin_id` in TOML must belong to your account |
| `Local server did not start` | `build.dev_command` must be ready within 30 s |
| `Port {port} is already in use` | Free the port or use `--port` |
| Data Bridge `Missing scope` / `Insufficient permissions` | Declare the scope, restart `whatalo dev`, reopen the plugin |
| `401` from your `/api` routes | Open via admin preview; verify with the current `WHATALO_CLIENT_SECRET` and `expectedAppId: app.id` |
| Every session token rejected | You compared `appId` with `WHATALO_CLIENT_ID` |
| `503`/missing API key in dev | Run `whatalo dev`; `env pull` does not supply `WHATALO_API_KEY` |
| Webhook `401` | Raw body, same secret on both sides, clock within 300 s |
| Empty webhook-driven data | Confirm `webhookUrl`, then `whatalo webhook trigger` and `whatalo logs` |
| Action ack `timeout` / `rate_limited` | Handle `success: false`; do not loop |
| Plugin fails to load / slow-load warning | Reduce initial bundle; host errors at 15 s |
| Environment issues | `whatalo doctor`, `whatalo info` |

Sources: [whatalo dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev#error-messages), [Data Bridge](https://developers.whatalo.com/docs/plugin-sdk/api-client/data-bridge#troubleshooting), [Utility Commands](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/utility-commands), independent example README (troubleshooting patterns consistent with the docs).

## Source conflicts

- Four internal portal links return 404: `/docs/plugin-sdk/publishing/review-process` (SDK overview; use `/docs/plugin-sdk/review-process`), `/docs/plugin-sdk/publishing/updates-and-versioning` (Distribution; use `/docs/plugin-sdk/updates-and-versioning`), `/docs/plugin-sdk/publishing` (Customers), and `/docs/plugin-sdk/cli-reference` (Verification; use `/docs/plugin-sdk/cli-reference/overview`).
- Updates & Versioning describes editing approved plugins in the portal; Deploy and Review Process describe deploying the manifest and submitting the saved candidate. Follow the deploy flow for manifest changes.
- Build Your First Plugin shows a validator summary with older dependency versions; trust the current command output.
