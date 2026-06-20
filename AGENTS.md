# Extension Agent Guide

> **Status**: Active
> **Updated**: 2026-06-19
> **Owner**: Studio extension maintainers
> **Purpose:** Extension-local supplements to the active Studio root `AGENTS.md`.

## Scope
- This file is not a replacement for the Studio root `AGENTS.md`.
- Extension development happens inside a Studio project, normally at `extensions/{EXTENSION_SLUG}/`.
- The extension manifest must exist at `extensions/{EXTENSION_SLUG}/.manifest` relative to the Studio project root.
- Read and follow the Studio root `AGENTS.md` first. Use this file only for extension-template-specific context and constraints.

## Template Constraints
- Keep this repository host-neutral, environment-neutral, and user-neutral.
- Do not add demo routes, demo templates, fake settings, sample providers, or placeholder runtime behavior that would execute when a new extension is activated.
- Placeholder files may explain where real extension code belongs, but they must not register visible UI, routes, providers, settings, data, or non-neutral translations.
- Replace template placeholders before release, including manifest metadata, extension slug, translation filenames/keys, preview assets, source URL, license owner, and README examples.

## Manifest
- `.manifest` is the extension identity source consumed by Studio.
- `EXTENSION_SLUG` defines the extension-owned namespace for identifiers, paths, translations, templates, assets, settings, provider keys, services, routes, storage keys, cache keys, and generated files.
- `EXTENSION_SLUG` must start with a lowercase letter, use only lowercase letters, digits, and single hyphen-separated segments, stay at 120 characters or less, and match the extension folder name exactly.
- Keep `.manifest`, README examples, translation keys, asset paths, and extension contributions aligned whenever extension identity or behavior changes.
- Use `EXTENSION_IMAGE` only for a real extension preview asset.
- Declare `EXTENSION_NAMESPACE` only when the extension ships PHP code under `src/`.

## Runtime Contributions
- `extension.php` is intentionally a no-op placeholder until a real extension needs runtime contributions.
- When used, `extension.php` may return only documented Studio contribution objects or providers after extension validation and activation.
- Do not perform work in `extension.php` beyond declaring contributions. In particular, do not directly include extension files, read or write files, spawn processes, open network sockets, inspect raw request/environment globals, or bypass documented extension points.
- Keep contribution IDs, setting keys, translation keys, route names, template references, service aliases, cache keys, and asset names extension-owned and collision-resistant.

## Optional Extension Docs
- `docs/CLASSMAP.md` is optional. If present, keep it current for extension-owned public callables, providers, commands, hooks, subscribers, Twig extensions, services, and other contributor-facing entry points.
- `docs/WORKLOG.md` is optional. If present, use it for extension-local session notes, verification, and follow-ups.
- If these files do not exist, do not create them only for process compliance; use README, CHANGELOG, issue notes, PR notes, or final notes as appropriate.

## Verification Note
- Run extension verification from the Studio project root whenever possible so root rules, host tooling, extension discovery, template namespaces, translation aggregation, and lifecycle checks are active.
