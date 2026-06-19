# Studio Package Template

> **Status**: Template
> **Purpose:** A neutral starting point and compact rulebook for developing Studio packages.

This repository is a template for packages that extend a Studio installation. A package is expected to live inside the host project at:

```text
/packages/{PACKAGE_SLUG}/
```

The package manifest must be available at:

```text
/packages/{PACKAGE_SLUG}/.manifest
```

Studio's root `AGENTS.md` and project rules remain binding during package development. This template documents the package-specific structure, contracts, constraints, and contribution patterns that should be followed in addition to the host rules.

## Package Goals

Packages should be small, inspectable extensions of Studio. They may provide UI, themes, modules, captcha providers, editor integrations, content tools, scheduled behavior, settings, hooks, assets, translations, or other documented contributions.

A good package should be:

- package-scoped and collision-resistant;
- installable, disableable, removable, and auditable through Studio's package lifecycle;
- explicit about the Studio contracts it uses;
- easy to review from `.manifest`, `package.php`, docs, translations, templates, and tests;
- conservative about dependencies, persistence, browser-visible metadata, and host integration assumptions.

## Required Layout

```text
packages/{PACKAGE_SLUG}/
├── .manifest
├── AGENTS.md
├── README.md
├── LICENSE
├── package.php
├── assets/
├── docs/
├── languages/
│   ├── de/
│   └── en/
├── src/
├── templates/
└── tests/
```

The directories may stay empty with `.gitkeep` files until the package needs them.

## Manifest

`.manifest` is the package identity source consumed by Studio.

Required fields:

```dotenv
PACKAGE_AUTHOR="Vendor Name"
PACKAGE_SLUG=package-template
PACKAGE_NAME="Package Template"
PACKAGE_VERSION=0.1.0
PACKAGE_SCOPE=module
```

Recommended metadata fields:

```dotenv
PACKAGE_DESCRIPTION="Short package description"
PACKAGE_DEPENDENCIES=[["system","0.2.6"]]
PACKAGE_LICENSE=MIT
PACKAGE_HOMEPAGE=https://example.com
PACKAGE_IMAGE=assets/preview.svg
PACKAGE_CHANNEL=main
PACKAGE_SOURCE=https://example.com/vendor/package-template
PACKAGE_DATE=2026-06-19
```

Guidelines:

- `PACKAGE_SLUG` is the technical owner namespace for package-owned identifiers.
- `PACKAGE_SCOPE` describes the host integration surface, such as `module`, `theme`, `captcha-provider`, `editor-extension`, or another documented Studio scope.
- `PACKAGE_DEPENDENCIES` declares package or system requirements. Keep versions honest and update them when host contracts change.
- `PACKAGE_IMAGE` points to an optional package preview image.
- `PACKAGE_NAMESPACE` may be declared when the package ships PHP under `src/`; PHP files must then use that namespace or one of its child namespaces.
- `PACKAGE_DATE` should reflect the packaged release date.
- Keep README examples, translation keys, asset paths, and `package.php` contributions aligned with the manifest.

## package.php

`package.php` returns an array of contribution objects consumed by Studio.

Typical contributions include:

- static view injection sets and routes;
- dynamic view injections for documented view slots;
- package settings with defaults, validation, labels, help text, and options;
- provider definitions for package scopes such as themes, captcha providers, editors, resolvers, or other host contracts;
- hook, event, command, scheduler, asset, or schema contributions once those contracts are documented by Studio.

Rules:

- Keep contributions declarative and easy to audit.
- Use package-owned IDs and keys, for example `{package-slug}.setting_name` or `pkg-{package-slug}-surface-name`.
- Use translated labels and help text for anything user-facing.
- Provide explicit defaults and validation for settings.
- Avoid broad side effects during package discovery. Loading `package.php` should describe contributions, not perform runtime work.
- Do not directly include files, read or write files, spawn processes, open network sockets, read raw environment/request globals, or bypass package extension points from `package.php`.

## Namespaces And Naming

Use the package slug consistently:

- translation keys: `pkg.{package-slug}.section.token`
- CSS selectors: `{package-slug}-component`
- JavaScript events: `{package-slug}:event-name`
- browser storage keys: `{package-slug}:key`
- route names: `pkg-{package-slug}-route-name`
- template/provider identifiers: package-owned and scope-specific
- cache keys and generated files: package-owned prefixes

Avoid `system`, `studio`, generic shared names, user-specific names, machine paths, or another package's namespace for package-owned behavior.

## Assets

`assets/` contains package frontend files and preview assets.

Recommended files:

- `assets/package.css`
- `assets/package.js`
- `assets/preview.svg`

Constraints:

- CSS must be package-scoped and avoid global resets or broad element selectors.
- JavaScript must be modular and clean up listeners, timers, observers, and pending async work.
- Asset filenames and DOM metadata must not leak secrets, answers, private state, or implementation hints for security-sensitive packages.
- Third-party assets must keep their license, notice, attribution, and source metadata with the package.
- Curated asset indexes must stay synchronized with actual files, declared counts, categories, families, checksums, and license metadata.

## Templates

`templates/` contains package Twig templates.

Rules:

- Keep templates small and package-scoped.
- Use lowercase, hyphenated filenames.
- Use deterministic translation keys for user-facing text.
- Provider templates should live under the path expected by the provider contract.
- Do not expose secrets, internal paths, private state, or answer labels in rendered markup.

Documented template roots:

- `templates/frontend/**` is referenced as `@frontend/...`.
- `templates/backend/**` is referenced as `@backend/...`.
- `templates/provider/{provider}/**` is referenced through matching provider slots.
- `templates/macros/{package-slug}/**` may contain package-owned macros.

## Translations

`languages/{locale}/` contains package translation catalogues.

Rules:

- Keep every supported locale synchronized.
- Use `pkg.{package-slug}.` as the key prefix.
- Translate labels, buttons, links, placeholders, help text, validation messages, flash messages, empty states, errors, navigation text, and accessibility text.
- Logs, developer exceptions, CLI output, test names, and internal debug strings do not need localization.
- Do not hardcode available languages in package runtime behavior.

## PHP Source

`src/` contains package PHP code when the package needs services, providers, value objects, validators, commands, hooks, or runtime behavior.

Rules:

- Use strict types and PSR-12.
- Prefer Studio and Symfony contracts over custom infrastructure.
- Keep classes small and behavior-focused.
- Validate public input at the boundary.
- Keep package-owned message codes, settings, service IDs, and cache keys under the package namespace.
- Avoid relying on undocumented Studio internals. If a host contract is missing, document the assumption and keep the package-side implementation narrow.

## Tests

`tests/` contains package-local tests when practical. Some integration tests may belong in the host Studio test suite if they require host package discovery, service wiring, routes, Live endpoints, or rendered surfaces.

Recommended coverage:

- package discovery and manifest parsing;
- `package.php` contribution shape;
- settings defaults and validation;
- provider registration;
- translation key coverage;
- rendered templates or provider output;
- package lifecycle behavior;
- security-sensitive token, replay, cache, or redaction behavior;
- asset index validation when curated assets are part of the package.

## Optional Docs

`docs/` may contain package-specific manuals, design notes, and maintenance files.

Optional files:

- `docs/CLASSMAP.md`: keep current when the package has public PHP callables, providers, commands, hooks, Twig extensions, event subscribers, or other contributor-facing entry points.
- `docs/WORKLOG.md`: keep current when the package uses a session worklog workflow.

If these files do not exist, do not create them only to satisfy process. Use README, CHANGELOG, issue notes, or PR notes for durable context instead.

## Security Constraints

Packages must treat request input and browser-visible state as hostile.

Follow these constraints:

- validate public inputs before using them in paths, selectors, route parameters, cache keys, storage keys, provider keys, commands, subprocesses, or response payloads;
- keep secrets in host configuration or installer-generated storage;
- do not log secrets, private state, sensitive generated output, host paths, environment details, or internal cache keys;
- use opaque identifiers for security-sensitive flows;
- consume one-shot IDs atomically where replay resistance matters;
- set appropriate cache headers for private Live/API responses;
- document cookies, browser storage, roles, ACLs, public endpoints, or retained data;
- provide uninstall, purge, and retention behavior for package-owned persisted state.

## Verification

Run verification from the Studio project root so package discovery and the root project rules are active.

Useful focused checks:

```bash
bin/lint packages/{PACKAGE_SLUG}
php -l packages/{PACKAGE_SLUG}/src/SomeFile.php
php bin/console lint:container
php bin/phpunit --filter PackageName
php bin/console render:route /some/package/route
```

Choose checks based on the changed surface:

- metadata/docs: lint Markdown and `.manifest`;
- translations: lint locale catalogues and compare keys;
- Twig: lint templates and render the affected host route or provider surface;
- CSS/JS: lint assets and run JavaScript tests or asset builds where applicable;
- PHP/service changes: syntax, focused tests, and container lint;
- lifecycle/provider changes: host package discovery/validation once available;
- curated assets: validate index syntax, path existence, declared counts, licenses, duplicates, and ignored-file status.

If a recommended check cannot run because the host contract or command does not exist yet, record that explicitly in PR notes or the final response.

## Release Checklist

Before release or PR readiness:

- `.manifest` matches package behavior, version, source, scope, and dependencies.
- `package.php` contributions are package-scoped and documented.
- README and package docs describe setup, behavior, constraints, and known host contracts.
- Translation catalogues are synchronized.
- License and third-party attribution files are present.
- Package-owned assets and indexes are synchronized.
- Tests or documented verification cover changed behavior.
- Security/privacy implications are reviewed.
- Install, update, deactivate, purge, and retained-state behavior are understood.
- Skipped checks and remaining follow-ups are recorded.
