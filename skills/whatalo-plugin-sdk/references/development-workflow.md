# Development workflow: preview, stop, clean

Summary only; read the linked pages for full contracts: [whatalo dev](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev), [whatalo dev clean](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/dev-clean), [CLI Overview](https://developers.whatalo.com/docs/plugin-sdk/cli-reference/overview), [Review Process](https://developers.whatalo.com/docs/plugin-sdk/review-process). Provenance: [source-map.md](source-map.md#development-workflow-sources).

## Compatibility check (required first)

```sh
whatalo dev clean --help
```

Follow the clean and preview-retention instructions below only if the installed CLI lists `dev clean` with the options described here. Otherwise follow the installed CLI's help and the `whatalo dev` page, and do not assume this behavior or infer a version.

## Scope of a dev preview

- `whatalo dev` previews on the development installation of the **selected development store** only.
- Merchants keep using the published URL while an update is in review; reviewers see the candidate.
- It does not overwrite the public manifest (`appUrl`, `webhookUrl`, pages, permissions) or a candidate under review. Published configuration changes only through `whatalo deploy` and review.
- An existing active or suspended installation on that store is left untouched; it is not converted into a preview.
- The development installation is created, updated, or revived from an uninstalled state as needed.

## Store selection

Used by `whatalo dev` and `whatalo dev clean`. Only development stores you own are eligible.

1. `--store <value>` / `-s <value>`: a store slug or public store ID. An invalid explicit value fails.
2. Cached selection, unless `--reset` is passed.
3. The only available development store.
4. Interactive prompt. Cancelling it does nothing.

`--reset` ignores the cache for that run (it does not delete it); an explicit `--store` still wins.

## Stopping a preview

`q` or `Ctrl+C` stops the local server and tunnel and keeps the preview installation. The preview is not necessarily offline: if it serves an uploaded frontend bundle, that frontend can keep loading while your local backend and tunnel are stopped. Ending the session retains the installation and any uploaded bundle. The request to terminate the remote development session at exit is best effort; it does not remove the preview. Use `whatalo dev clean` to explicitly clean the selected store's preview.

## `whatalo dev clean`

```sh
whatalo dev clean --store my-dev-store
whatalo dev clean --store my-dev-store --json
```

| Option | Purpose |
| --- | --- |
| `--store <value>`, `-s` | Store slug or public store ID |
| `--reset` | Ignore the cached store selection |
| `--portal-url <url>` | Developer Portal URL |
| `--json` | JSON output only; it can still prompt, so pass `--store` in automation |

Outcomes for the selected store:

- `restored`: the plugin is approved or has a live public version (including while an update is in review). The preview URL and manifest are cleared, the development installation's granted scopes and its linked API key's scopes are reset to the approved scopes, and the installation keeps its development status. Billing and consent are not affected.
- `removed`: the plugin has no published version and is not approved; the preview installation is removed.
- `noop`: there is no development installation, or there is nothing to clean.

Rules:

- After `restored`, re-test any behavior that depends on scopes; other installation state is not reset.
- Do not expect restoration of an older published version.
- JSON: `{ "status": "restored" | "removed" | "noop", "storePublicId": "<id>", "storeName": "<display name>", "plugin": "<slug>" }`. `storePublicId` is accepted by `--store`.
- Errors stay human-readable even with `--json`; handled cleanup failures exit with code 1.
- Never changes merchant active or suspended installations, other stores, or the marketplace listing. There is no option to clean every store.

## Embedded UI

Unchanged by this workflow: host-managed scroll with `html, body { overflow: hidden; background: transparent; }` plus root `useAutoResize()` and `useThemeSync()`. No intermediate scroll container or wildcard workarounds. See [ui-contract.md](ui-contract.md#scroll-and-resize).
