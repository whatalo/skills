# Whatalo Skills

Agent Skills that help coding agents build on the Whatalo platform.

## Status

This repository is in its foundation stage. **There are no installable skills yet.**

The first planned skill is `whatalo-plugin-sdk`, covering the public Whatalo plugin SDK. It will be published only after its guidance is verified against the current public SDK documentation and a license has been selected for this repository.

## Planned installation

> **Future command — not available yet.** It will not work until the first skill is published.

```sh
npx skills add https://github.com/whatalo/skills
```

## Layout

```text
skills/
└── <skill-name>/
    ├── SKILL.md      # Required: the skill's source of truth
    ├── references/   # Optional: supporting docs
    ├── assets/       # Optional: templates and examples
    └── scripts/      # Optional: helper scripts
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Agents working in this repository should follow [AGENTS.md](AGENTS.md).

## License

Not yet selected. See [CONTRIBUTING.md](CONTRIBUTING.md#license).
