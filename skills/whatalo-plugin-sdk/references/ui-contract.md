# UI contract: tokens, layout, mobile, theme, resize, accessibility

Canonical source: the `react-vite` starter in `create-whatalo-plugin@1.6.0` (`src/components/whatalo-ui/`). The public UI pages describe the same library but several examples use props and token values that the shipped starter does not have (see "Source conflicts"). Prescribe only what the starter ships.

## Policy

The public docs say the components live in your project and you "can modify any component or extend the design tokens freely" ([UI Components Overview](https://developers.whatalo.com/docs/plugin-sdk/ui-components/overview#customization)). The maintainer established a stricter approval policy, stated in `SKILL.md`: starter components and existing `--wui-*` tokens and `.wui-*` classes only, no alternate design systems. Semantic HTML inside those components is allowed for accessibility. This is maintainer policy, not a description of platform review automation.

## Components and real props

Import from `./components/whatalo-ui` (index exports 16 components and 2 hooks; `Layout.Section`, `List.Item`, and `Accordion.Item` are sub-components).

| Component | Props (defaults) |
| --- | --- |
| `Page` | `width?: "narrow" \| "default" \| "full"` (`default`); renders `<main class="wui-page">` |
| `PageHeader` | `title`, `subtitle?`, `children?` (actions slot), `backAction?: { url, label? }`; title renders as `<h1>` |
| `Layout` | `columns?: "default" \| "halves"` |
| `Layout.Section` | `variant?: "full" \| "oneThird" \| "oneHalf"`; adds class names only — `full` adds none and the `one-third`/`one-half` classes have no CSS rules, so placement comes from `Layout` |
| `Card` | `padding?: "none" \| "tight" \| "default" \| "loose"` |
| `Box` | `as?: "div" \| "span" \| "section"`, `padding?: "100"…"500"`, `background?: "default" \| "surface" \| "surface-active" \| "transparent"`, `borderWidth?: "none" \| "thin"`, `borderRadius?: "none" \| "100" \| "200" \| "300"`, `borderColor?: "default" \| "subdued"` |
| `BlockStack` | `gap?: "100"…"500"` (`300`), `align?: "start" \| "center" \| "end"` |
| `InlineStack` | `gap?` (`300`), `align?: "start" \| "center" \| "end" \| "space-between"`, `wrap?` (`true`) |
| `Divider` | none |
| `Text` | `variant?: "headingLg" \| "headingMd" \| "headingSm" \| "bodyLg" \| "bodyMd" \| "bodySm" \| "caption"` (`bodyMd`), `as?: "h1"…"h4" \| "p" \| "span"`, `color?: "default" \| "subdued" \| "success" \| "critical"`, `fontWeight?: "regular" \| "medium" \| "semibold" \| "bold"` |
| `Badge` | `tone?: "info" \| "success" \| "warning" \| "critical" \| "neutral"` (`neutral`) |
| `Banner` | `title?`, `tone?: "info" \| "success" \| "warning" \| "critical"` (`info`), `onDismiss?` |
| `List` / `List.Item` | `type?: "bullet" \| "number"` (`bullet`) |
| `Spinner` | `size?: "small" \| "default" \| "large"` |
| `Button` | `variant?: "primary" \| "secondary" \| "plain" \| "destructive"` (`secondary`), `size?: "slim" \| "medium" \| "large"`, `loading?`, `url?`, `external?`, `fullWidth?`, plus native button attributes except `className` |
| `Link` | `url` (required), `external?`, `removeUnderline?`, `monochrome?` |
| `Accordion` / `Accordion.Item` | `Accordion.Item`: `title`, `defaultOpen?` |

`gap` and `padding` values map to `--wui-space-100…500`. The stylesheet also defines `.wui-info-grid` (key/value `dl`) without a component.

## Tokens (`styles.css`)

Token names only; read values from the starter file, never copy values from the docs pages.

- Surfaces: `--wui-bg`, `--wui-surface`, `--wui-surface-active`, `--wui-surface-hover`
- Text: `--wui-text`, `--wui-text-subdued`, `--wui-text-success`, `--wui-text-critical`
- Borders: `--wui-border`, `--wui-border-subdued`
- Interactive: `--wui-primary`, `--wui-primary-hover`, `--wui-primary-text`, `--wui-secondary`, `--wui-secondary-hover`, `--wui-interactive`, `--wui-interactive-hover`
- Tones: `--wui-{info,success,warning,critical}-{bg,text,border}`, `--wui-neutral-bg`, `--wui-neutral-text`
- Spacing: `--wui-space-100` (4px) … `--wui-space-500` (20px)
- Radius: `--wui-radius-100/200/300`, `--wui-radius`, `--wui-radius-sm`
- Typography: `--wui-font`, `--wui-font-size-heading-{lg,md,sm}`, `--wui-font-size-body-{lg,md,sm}`, `--wui-font-size-caption`
- Page widths: `--wui-page-narrow` (680px), `--wui-page-default` (960px)

Light values live in `:root`; dark values in `:root[data-theme="dark"]` (zinc-based, with `color-scheme: dark`). Spacing, radius, typography, and widths are defined only in `:root`.

## Layout and mobile

- Every page: `Page` → `PageHeader` → content. `PageHeader` wraps (`flex-wrap`) and puts `children` in an actions area.
- `Page` is centered with `max-width: var(--wui-page-default)` and `padding: var(--wui-space-500)`; `width="narrow"` uses 680px, `width="full"` uses 100%.
- `Layout` is one column by default and becomes `2fr 1fr` at `min-width: 769px`; `columns="halves"` becomes `1fr 1fr` at the same breakpoint. Grid position follows source order: put the main section first and the secondary section second, as the starter does. `Layout.Section` variants do not change layout by themselves.
- `InlineStack` wraps by default; use it for button groups and label/value rows. Prefer `BlockStack` for primary content in narrow iframes.
- Format dates and numbers with `bridge.locale`; format money with the `currency` returned by the data.

Sources: starter `styles.css`, `app.tsx`, `pages/settings.tsx`; [Design Guidelines](https://developers.whatalo.com/docs/plugin-sdk/best-practices/design-guidelines).

## Theme

1. `src/main.tsx` imports `./theme-bootstrap` first; it reads `?whatalo_theme=light|dark` and sets `data-theme` and `colorScheme` before render.
2. `useThemeSync()` reads `whatalo_theme` from the URL (the starter comments say this covers host modals, where the bridge may not be ready at render time) and otherwise `useWhataloContext().theme`, then sets `document.documentElement` `data-theme` and `style.colorScheme`.
3. The SDK bridge also sets `data-theme` when a `whatalo:context` message arrives, and the admin re-sends context when its theme changes.

Static inspection shows the URL theme hint takes precedence over bridge context in `useThemeSync()`, so live theme toggling is not guaranteed. Test toggling the admin theme at runtime; if the plugin does not follow, report the mismatch instead of inventing a fix.

Use tokens for every color so both themes work. Sources: starter files; SDK 1.5.0 bridge; [Theme Integration](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/theme-integration).

## Scroll and resize

- Base CSS: `html, body { overflow: hidden; background: transparent; }`. The admin scrolls the page; the background comes from the admin. Do not set `var(--wui-bg)` on the body (starter comment).
- `useAutoResize()` measures `document.documentElement.scrollHeight` on first frame and on every `ResizeObserver` change of `document.body`, and calls `resize(height)` only when the change exceeds 1px.
- Bridge `resize` limits: 30 calls per 10 seconds; height constraints documented as min 200px, max 15000px, default 400px. Do not call `resize` in loops or animations, and do not hard-code iframe heights.
- The starter CSS defines a thin `html` scrollbar style whose comment mentions modal views; the starter ships no script that applies a modal mode. Modal scroll behavior is unverified; check the installed SDK and docs before relying on it.
- The host shows "Loading plugin..." at 3 s, a slow-load warning at 8 s, and an error at 15 s; keep the first render light and lazy-load heavy content.

Sources: [Hooks](https://developers.whatalo.com/docs/plugin-sdk/ui-components/hooks), [UI Actions](https://developers.whatalo.com/docs/plugin-sdk/app-bridge/ui-actions#resize), [Performance](https://developers.whatalo.com/docs/plugin-sdk/best-practices/performance).

## States and feedback

- Loading: the starter's `App` returns `null` until `context.isReady`; use `Spinner` (or `Button loading`) while data loads.
- Empty: explain why and suggest a next step, for example with `Banner tone="info"`.
- Error: `Banner tone="critical"` or `Text color="critical"`; keep the UI responsive.
- Success: `bridge.toast.show(title, { variant: "success" })`.

## Accessibility (as shipped)

Keep these starter behaviors when composing pages:

- `Spinner`: `role="status"`, `aria-label="Loading"`.
- `Banner`: `role="status"`; dismiss button has `aria-label="Dismiss"`.
- `Accordion.Item`: real `<button type="button">` with `aria-expanded`.
- `Button`: defaults to `type="button"`, disabled while `loading`; `url` renders an anchor.
- `Link`/`Button` with `external`: `target="_blank" rel="noopener noreferrer"`.
- `Text` variants render semantic tags (`headingLg`→`h1`, `headingMd`→`h2`, `headingSm`→`h3`); use `as` to keep heading order.
- Use clear, action-oriented labels ("Save changes", not "Submit").

## Source conflicts

| Docs pages show | Starter 1.6.0 ships |
| --- | --- |
| `Badge variant="success\|warning\|error"`, `Banner status="..."`, `status="error"` | `tone`, with `critical` instead of `error` |
| `Text variant="heading"` / `"subdued"` | `variant="headingMd"` etc., `color="subdued"` |
| `PageHeader description` | `subtitle` |
| `Link href` | `url` |
| `<Accordion title>` | `Accordion` wrapping `Accordion.Item title` |
| Theme query parameter `?theme=` | `?whatalo_theme=` |
| Token values such as `--wui-bg: #ffffff`, `--wui-primary: #6366f1`, `--wui-radius: 8px`, dark selector `[data-theme="dark"]` | Different values (for example `--wui-bg: #f6f6f7`, `--wui-radius: 12px`) and selector `:root[data-theme="dark"]` |
| "Seventeen components" | Index exports 16 components plus `Layout.Section`, `List.Item`, `Accordion.Item` |
| `const { currentPage } = useAppBridge()` | `useAppBridge()` in SDK 1.5.0 does not return `currentPage`; use `useWhataloContext().currentPage` (as the starter does) |
| UI Actions: out-of-range heights are "clamped silently" | Same page lists `"out_of_bounds"` as a resize error; handle a failed ack |
