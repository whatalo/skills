# Whatalo Skills

Agent Skills, editor rules, and plugin manifests that help coding agents build on the Whatalo platform.

## Status

This repository is in its foundation stage. **There are no installable skills or plugin manifests yet.**

The first planned skill is `whatalo-plugin-sdk`, covering the public Whatalo plugin SDK. It will be published only after its guidance is verified against the current public SDK documentation and a license has been selected for this repository.

## Planned installation

> **Future command — not available yet.** It will not work until the first skill is published.

```sh
npx skills add https://github.com/whatalo/skills
```

Host-specific plugin installation will be documented once the corresponding manifests exist.

## Layout

The repository is designed as a single `whatalo` plugin distributed to several agent hosts:

- `skills/<skill-name>/SKILL.md` with `references/` — skills and their supporting docs.
- `rules/<product>.mdc` — editor rules.
- `plugin.json`, `.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.agents/plugins/` — plugin and marketplace manifests per host.
- `.mcp.json`, `mcp.json` — MCP server configuration.
- `.github/workflows/` — security scanning.

Most of these paths are reserved but not yet created. See [AGENTS.md](AGENTS.md) for the complete target tree, each file's role, and the current materialization status.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Agents working in this repository should follow [AGENTS.md](AGENTS.md).

## License

Not yet selected. See [CONTRIBUTING.md](CONTRIBUTING.md#license).
