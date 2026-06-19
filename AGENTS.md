# Package Agent Guide

> **Status**: Active
> **Updated**: 2026-06-19
> **Owner**: Studio package maintainers
> **Purpose:** Package-local supplements to the active Studio root `AGENTS.md`.

## Scope
- This file is not a replacement for the Studio root `AGENTS.md`.
- Package development happens inside a Studio project, normally at `packages/{PACKAGE_SLUG}/`.
- The package manifest must exist at `packages/{PACKAGE_SLUG}/.manifest` relative to the Studio project root.
- Read and follow the Studio root `AGENTS.md` first. Use this file only for package-template-specific context and constraints.

## Template Constraints
- Keep this repository host-neutral, environment-neutral, and user-neutral.
- Do not add demo routes, demo templates, fake settings, sample providers, or placeholder runtime behavior that would execute when a new package is activated.
- Placeholder files may explain where real package code belongs, but they must not register visible UI, routes, providers, settings, data, or non-neutral translations.
- Replace template placeholders before release, including manifest metadata, package slug, translation filenames/keys, preview assets, source URL, license owner, and README examples.

## Manifest
- `.manifest` is the package identity source consumed by Studio.
- `PACKAGE_SLUG` defines the package-owned namespace for identifiers, paths, translations, templates, assets, settings, provider keys, services, routes, storage keys, cache keys, and generated files.
- Keep `.manifest`, README examples, translation keys, asset paths, and package contributions aligned whenever package identity or behavior changes.
- Use `PACKAGE_IMAGE` only for a real package preview asset.
- Declare `PACKAGE_NAMESPACE` only when the package ships PHP code under `src/`.

## Runtime Contributions
- `package.php` is intentionally a no-op placeholder until a real package needs runtime contributions.
- When used, `package.php` may return only documented Studio contribution objects or providers after package validation and activation.
- Do not perform work in `package.php` beyond declaring contributions. In particular, do not directly include package files, read or write files, spawn processes, open network sockets, inspect raw request/environment globals, or bypass documented package extension points.
- Keep contribution IDs, setting keys, translation keys, route names, template references, service aliases, cache keys, and asset names package-owned and collision-resistant.

## Optional Package Docs
- `docs/CLASSMAP.md` is optional. If present, keep it current for package-owned public callables, providers, commands, hooks, subscribers, Twig extensions, services, and other contributor-facing entry points.
- `docs/WORKLOG.md` is optional. If present, use it for package-local session notes, verification, and follow-ups.
- If these files do not exist, do not create them only for process compliance; use README, CHANGELOG, issue notes, PR notes, or final notes as appropriate.

## Verification Note
- Run package verification from the Studio project root whenever possible so root rules, host tooling, package discovery, template namespaces, translation aggregation, and lifecycle checks are active.
