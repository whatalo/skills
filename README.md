# Whatalo Skills

Agent Skills that help coding agents build on the Whatalo platform.

## Skills

| Skill | Covers |
| --- | --- |
| [`whatalo-plugin-sdk`](skills/whatalo-plugin-sdk/SKILL.md) | Building Whatalo admin plugins with `@whatalo/plugin-sdk` and the `whatalo` CLI: setup, manifest, UI contract, App Bridge, Data Bridge, session tokens, REST client, webhooks, billing, development preview, deploy, and review. |

The skill routes agents to the official [Whatalo Plugin SDK documentation](https://developers.whatalo.com/docs/plugin-sdk) and constrains how they apply it; it does not replace the docs. It was verified on 2026-10-09 against the public documentation and `@whatalo/plugin-sdk` 1.5.0, the CLI family (`whatalo`, `@whatalo/cli`, `@whatalo/cli-kit`, `create-whatalo-plugin`) 1.7.0, with the full package inspection done on 1.6.0 and the 1.7.0 changes diffed. See [source-map.md](skills/whatalo-plugin-sdk/references/source-map.md) for the sources behind each reference.

## Installation

Install the skill with the [`skills`](https://www.npmjs.com/package/skills) CLI:

```sh
npx skills add whatalo/skills --skill whatalo-plugin-sdk
```

List the skills available in this repository:

```sh
npx skills add whatalo/skills --list
```

Add `-g` to install for your user instead of the current project:

```sh
npx skills add whatalo/skills --skill whatalo-plugin-sdk -g
```

Update installed skills:

```sh
npx skills update
```

## Layout

- `skills/<skill-name>/SKILL.md` — the skill entry point.
- `skills/<skill-name>/references/` — topic references the skill links to.

See [AGENTS.md](AGENTS.md) for the full repository structure and authoring rules.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Agents working in this repository should follow [AGENTS.md](AGENTS.md).

## License

[Apache License 2.0](LICENSE).
