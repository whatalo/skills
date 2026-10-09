# Agent Instructions

Rules for coding agents working in this repository.

## Public-only content

- This repository is public. Never write secrets, credentials, tokens, account or tenant IDs, personal data, internal architecture, private URLs, or logs.
- Base skill guidance only on current, publicly available Whatalo SDK documentation that you have verified.

## Skill structure

- The source of truth for a skill is `skills/<skill-name>/SKILL.md`.
- Optional supporting files go in `skills/<skill-name>/references/`, `assets/`, or `scripts/`.
- Do not create placeholder or stub skills that could be installed.
- Do not duplicate a skill; update the existing one.

## Accuracy

- State the SDK version range each skill targets.
- Do not promise behavior that has not shipped.
- If public docs lack required information, report the gap instead of inventing guidance.

## Publication gates

- No skill may be published until a repository license is selected by the maintainers. Do not add or choose a license yourself.
- Add plugin manifests, MCP configuration, workflows, or package metadata only when a real integration exists.

## Validation

- Do not run `verify:ci` in this repository or borrow application CI commands from another repository.
- For documentation-only changes, review content, relative links, privacy, and whitespace.

## Registered skills

None yet.
