# Contributing

Skills here should stay small and point agents to the right public documentation rather than restate it.

## Before proposing a skill

- Confirm no existing skill already covers the topic. Extend it instead of adding a duplicate.
- Verify every instruction against the **current public** Whatalo SDK documentation. A link that resolves is not proof; read the page.
- State which SDK versions the guidance applies to. Do not describe unreleased or unshipped behavior as guaranteed.
- If public documentation is missing something the skill needs, describe the gap in the pull request instead of filling it with unverified guidance.

## Repository layout

The complete target structure — skills, rules, plugin manifests, MCP configuration, and workflows — and the status of each path are defined in [AGENTS.md](AGENTS.md#target-structure). Follow it instead of inventing new locations.

Each skill lives in `skills/<skill-name>/`:

- `SKILL.md` — required; the single source of truth for the skill.
- `references/` — optional supporting material, as flat `<topic>.md` files or nested `<topic>/` directories.

Keep `SKILL.md` concise and move long examples or background into references.

## Public content only

This repository is public. Never include secrets, credentials, account or tenant IDs, customer data, internal architecture details, logs, or private URLs.

## Integrations

Plugin and marketplace manifests, MCP configuration, the security workflow, `CODEOWNERS`, and brand assets are part of the target structure. Each is created only when its gate in [AGENTS.md](AGENTS.md#materialization-gates) is met. Do not add empty, disabled, or speculative versions to fill the tree.

## License

A license has not been selected yet. The maintainers will choose one before the first skill is published. Until then, please open an issue before contributing substantial content.
