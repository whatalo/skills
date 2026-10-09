# Agent Instructions

Rules for coding agents working in this repository. This file is the structure guide for all future authoring: it defines the complete target tree, what each file is responsible for, and the gates a file must pass before it is materialized.

## Public-only content

- This repository is public. Never write secrets, credentials, tokens, account or tenant IDs, personal data, internal architecture, private URLs, or logs.
- Base skill guidance only on current, publicly available Whatalo SDK documentation that you have verified.
- Write original Whatalo guidance. Do not include external inspiration credits, copied branding, provider endpoints, or third-party assets. Factual names of supported agent integrations are allowed. Never copy licensed material while removing required notices.

## Target structure

The repository is one plugin, `whatalo`, distributed to several agent hosts. Every host manifest points at the same `skills/` directory and the same MCP configuration. The complete target tree is:

```text
.
├── .agents/
│   └── plugins/
│       └── marketplace.json         # Generic agent marketplace listing
├── .claude-plugin/
│   ├── marketplace.json             # Claude Code marketplace listing
│   └── plugin.json                  # Claude Code plugin manifest
├── .codex-plugin/
│   └── plugin.json                  # Codex plugin manifest and UI metadata
├── .cursor-plugin/
│   ├── marketplace.json             # Cursor marketplace listing
│   └── plugin.json                  # Cursor plugin manifest
├── .github/
│   └── workflows/
│       └── semgrep.yml              # Static security scan
├── .gitignore
├── .mcp.json                        # MCP servers for host-specific manifests
├── AGENTS.md                        # This file
├── CODEOWNERS
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── icon.png                         # Raster plugin icon
├── logo.svg                         # Vector plugin logo
├── mcp.json                         # MCP servers in the host-neutral schema
├── plugin.json                      # Host-neutral plugin manifest
├── rules/
│   └── <product>.mdc                # Editor rule, e.g. whatalo-plugin-sdk.mdc
└── skills/
    └── <skill-name>/                # First skill: whatalo-plugin-sdk
        ├── SKILL.md                 # Required: the skill's source of truth
        └── references/
            ├── <topic>.md           # Flat topic reference
            └── <topic>/             # Or a nested topic directory
                ├── README.md        # Topic overview and index
                ├── api.md
                ├── configuration.md
                ├── gotchas.md
                └── patterns.md
```

`odd/`, `.atl/`, and `.codegraph/` are local tooling state, ignored by `.gitignore`. Never document them as part of the product or commit them.

## File responsibilities

### Plugin manifests

All manifests describe the same plugin. Keep shared identity and release metadata consistent wherever each host schema supports those fields; change them together in one commit. The fields below describe the target integration layout, not a universal schema: verify each host's current official schema before creating its manifest.

| File | Role | Key fields |
| --- | --- | --- |
| `plugin.json` | Host-neutral plugin manifest. | `$schema` (public plugin schema), `name`, `version`, `description`, `author`, `homepage`, `repository`, `license`, `keywords`. |
| `.claude-plugin/plugin.json` | Claude Code plugin manifest. | Common fields plus `mcpServers: "./.mcp.json"`. |
| `.claude-plugin/marketplace.json` | Lists the plugin in a Claude Code marketplace. | `$schema`, `name`, `description`, `metadata`, `owner`, `plugins[]` with `source: "./"` and the common fields. |
| `.codex-plugin/plugin.json` | Codex plugin manifest. | Common fields plus `skills: "./skills/"`, `mcpServers: "./.mcp.json"`, and `interface` (`displayName`, `shortDescription`, `longDescription`, `developerName`, `category`, `capabilities`, `websiteURL`, `privacyPolicyURL`, `termsOfServiceURL`, `defaultPrompt[]`, `brandColor`, `composerIcon`, `logo`, `screenshots[]`). |
| `.cursor-plugin/plugin.json` | Cursor plugin manifest. | Common fields plus `logo: "logo.svg"`. |
| `.cursor-plugin/marketplace.json` | Lists the plugin in a Cursor marketplace. | `name`, `owner`, `plugins[]` with `source: "./"` and `description`. |
| `.agents/plugins/marketplace.json` | Generic agent marketplace listing installed from the Git URL. | `name`, `interface.displayName`, `plugins[]` with `source` (`source: "url"`, `url`), `policy` (`installation`, `authentication`), `category`. |

### MCP configuration

| File | Role |
| --- | --- |
| `.mcp.json` | `mcpServers` map referenced by the Claude Code and Codex manifests. HTTP servers use `type: "http"` and `url`. |
| `mcp.json` | Same servers in the host-neutral schema (`$schema`, `type: "streamable-http"`). |

Both files must list the same servers. Only add them when Whatalo publishes a public MCP endpoint; never point them at private, staging, or guessed URLs.

### Repository files

| File | Role |
| --- | --- |
| `.github/workflows/semgrep.yml` | Static security scan on pull requests, pushes to the default branch, a schedule, and manual dispatch. Read-only `contents` permission, pinned tool version. |
| `.gitignore` | Excludes OS, editor, environment, and local agent/tooling state. |
| `CODEOWNERS` | Review ownership. Uses maintainer handles chosen by the Whatalo maintainers only. |
| `CONTRIBUTING.md` | Human contribution rules: verification, linking, public-only content. |
| `LICENSE` | Repository license. Selected by the maintainers only. |
| `README.md` | Human overview, installation per host, and status. |
| `icon.png`, `logo.svg` | Official Whatalo brand assets referenced by manifests. Supplied by the maintainers only. |

### Rules

`rules/<product>.mdc` holds an editor rule for one product (for example `rules/whatalo-plugin-sdk.mdc`). It starts with front matter containing `description` and `alwaysApply: false`, then gives short, retrieval-first guidance that points to the public Whatalo documentation.

### Skills

- The source of truth for a skill is `skills/<skill-name>/SKILL.md`. It starts with front matter containing `name` (matching the directory) and `description` (what the skill covers and when to use it).
- The first planned skill is `whatalo-plugin-sdk`, targeting the public Whatalo plugin SDK.
- Supporting material goes in `skills/<skill-name>/references/`, either as flat `<topic>.md` files or as nested `<topic>/` directories with `README.md`, `api.md`, `configuration.md`, `gotchas.md`, and `patterns.md` (add other topic files only when needed).
- Keep `SKILL.md` concise and link to references and public documentation instead of restating them.
- Do not duplicate a skill; update the existing one.

## Materialization status

Only files that exist with real content count as done. Current inventory:

| Path | Status |
| --- | --- |
| `AGENTS.md`, `README.md`, `CONTRIBUTING.md`, `.gitignore` | Materialized. |
| `LICENSE` | Materialized. Apache License 2.0, selected by the maintainers. |
| `skills/whatalo-plugin-sdk/SKILL.md`, `skills/whatalo-plugin-sdk/references/*.md` | Materialized. Summarizes public docs; inspected `@whatalo/plugin-sdk` 1.5.0, `whatalo` and `create-whatalo-plugin` 1.6.0 on 2026-10-09; `references/development-workflow.md` is verified against the maintainers' implementation and public docs, not the npm inspection. Published under Apache-2.0; installable with `npx skills add whatalo/skills --skill whatalo-plugin-sdk`. |
| `.agents/plugins/`, `.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.github/workflows/`, `rules/` | Directory reserved with `.gitkeep` only. No manifest, workflow, or rule exists yet. |
| All plugin manifests, `.mcp.json`, `mcp.json` | Not created. Blocked on the gates below. |
| `.github/workflows/semgrep.yml` | Not created. Blocked on maintainer approval. |
| `CODEOWNERS` | Not created. Blocked on maintainer handles. |
| `icon.png`, `logo.svg` | Not created. Blocked on official brand assets. |
| `rules/<product>.mdc` | Not created. Blocked on verified public SDK documentation. |

Update this table in the same change that materializes or removes a file.

## Materialization gates

Create a target file only when its gate is met. Never create placeholder, empty, or disabled versions of these files to fill the tree; use `.gitkeep` to reserve a directory instead.

- **Skills, references, rules**: every statement verified against the current public Whatalo SDK documentation, with the targeted SDK version range stated. If the docs lack required information, report the gap instead of inventing guidance. Never create an installable skill without real content.
- **Plugin manifests**: at least one real skill exists and the brand assets exist (the license, Apache-2.0, is selected). `version` is shared and bumped together across all manifests.
- **MCP configuration**: a public, documented Whatalo MCP endpoint exists.
- **Workflow**: approved by the maintainers; no secrets in the workflow file.
- **`LICENSE`, `CODEOWNERS`, `icon.png`, `logo.svg`**: supplied or selected by the maintainers. Agents never choose a license, invent owners, or draw logos.
- **Publication**: skills are published under Apache-2.0. New skill front matter declares `license: Apache-2.0`.

## Validation

- Do not run `verify:ci` in this repository or borrow application CI commands from another repository.
- For documentation-only changes, review content, relative links, privacy, and whitespace.
- For JSON manifests, check that each file parses, that shared fields match across all manifests, and that every referenced path (`./skills/`, `./.mcp.json`, `logo.svg`) exists.
- For skills and rules, check front matter, that `name` matches the directory, and that relative links resolve.
- Search changes for secrets, private URLs, and unrelated external branding before finishing.

## Registered skills

| Skill | Trigger | Path |
| --- | --- | --- |
| `whatalo-plugin-sdk` | Whatalo plugin, `@whatalo/plugin-sdk`, `whatalo` CLI, App Bridge, `whatalo-ui`, plugin webhooks or billing | [skills/whatalo-plugin-sdk/SKILL.md](skills/whatalo-plugin-sdk/SKILL.md) |
