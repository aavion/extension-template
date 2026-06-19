# Studio Extension Template

> **Status**: Template
> **Purpose:** A neutral starting point and compact rulebook for developing Studio extensions.

This repository is a template for extensions that extend a Studio installation. An extension is expected to live inside the host project at:

```text
/extensions/{EXTENSION_SLUG}/
```

The extension manifest must be available at:

```text
/extensions/{EXTENSION_SLUG}/.manifest
```

Studio's root `AGENTS.md` and project rules remain binding during extension development. This template documents the extension-specific structure, contracts, constraints, and contribution patterns that should be followed in addition to the host rules.

## Extension Goals

Extensions should be small, inspectable extensions of Studio. They may provide UI, themes, modules, captcha providers, editor integrations, content tools, scheduled behavior, settings, hooks, assets, translations, or other documented contributions.

A good extension should be:

- extension-scoped and collision-resistant;
- installable, disableable, removable, and auditable through Studio's extension lifecycle;
- explicit about the Studio contracts it uses;
- easy to review from `.manifest`, `extension.php`, docs, translations, templates, and tests;
- conservative about dependencies, persistence, browser-visible metadata, and host integration assumptions.

## Required Layout

```text
extensions/{EXTENSION_SLUG}/
├── .manifest
├── AGENTS.md
├── README.md
├── LICENSE
├── extension.php
├── assets/
├── docs/
├── languages/
│   ├── de/
│   └── en/
├── src/
├── templates/
└── tests/
```

The directories may stay empty with `.gitkeep` files until the extension needs them.

## Manifest

`.manifest` is the extension identity source consumed by Studio.

Required fields:

```dotenv
EXTENSION_AUTHOR="Vendor Name"
EXTENSION_SLUG=extension-template
EXTENSION_NAME="Extension Template"
EXTENSION_VERSION=0.1.0
EXTENSION_SCOPE=module
```

Recommended metadata fields:

```dotenv
EXTENSION_DESCRIPTION="Short extension description"
EXTENSION_DEPENDENCIES=[["system","0.2.6"]]
EXTENSION_LICENSE=MIT
EXTENSION_HOMEPAGE=https://example.com
EXTENSION_IMAGE=assets/preview.svg
EXTENSION_CHANNEL=main
EXTENSION_SOURCE=https://example.com/vendor/extension-template
EXTENSION_DATE=2026-06-19
```

Guidelines:

- `EXTENSION_SLUG` is the technical owner namespace for extension-owned identifiers.
- `EXTENSION_SCOPE` describes the host integration surface, such as `module`, `frontend-theme`, `backend-theme`, `system-template`, `captcha-provider`, `editor-provider`, or another documented Studio scope.
- `EXTENSION_DEPENDENCIES` declares extension or system requirements. Keep versions honest and update them when host contracts change.
- `EXTENSION_IMAGE` points to an optional extension preview image.
- `EXTENSION_NAMESPACE` may be declared when the extension ships PHP under `src/`; PHP files must then use that namespace or one of its child namespaces.
- `EXTENSION_DATE` should reflect the extension's release date.
- Keep README examples, translation keys, asset paths, and `extension.php` contributions aligned with the manifest.

## extension.php

`extension.php` returns an array of contribution objects consumed by Studio.

Typical contributions include:

- static view injection sets and routes;
- dynamic view injections for documented view slots;
- extension settings with defaults, validation, labels, help text, and options;
- provider definitions for extension scopes such as themes, captcha providers, editors, resolvers, or other host contracts;
- hook, event, command, scheduler, asset, or schema contributions once those contracts are documented by Studio.

Rules:

- Keep contributions declarative and easy to audit.
- Use extension-owned IDs and keys, for example `{extension-slug}.setting_name` or `ext-{extension-slug}-surface-name`.
- Use translated labels and help text for anything user-facing.
- Provide explicit defaults and validation for settings.
- Avoid broad side effects during extension discovery. Loading `extension.php` should describe contributions, not perform runtime work.
- Do not directly include files, read or write files, spawn processes, open network sockets, read raw environment/request globals, or bypass extension points from `extension.php`.

## Namespaces And Naming

Use the extension slug consistently:

- translation keys: `ext.{extension-slug}.section.token`
- CSS selectors: `{extension-slug}-component`
- JavaScript events: `{extension-slug}:event-name`
- browser storage keys: `{extension-slug}:key`
- route names: `ext-{extension-slug}-route-name`
- template/provider identifiers: extension-owned and scope-specific
- cache keys and generated files: extension-owned prefixes

Avoid `system`, `studio`, generic shared names, user-specific names, machine paths, or another extension's namespace for extension-owned behavior.

## Assets

`assets/` contains extension frontend files and preview assets.

Recommended files:

- `assets/extension.css`
- `assets/extension.js`
- `assets/preview.svg`

Constraints:

- CSS must be extension-scoped and avoid global resets or broad element selectors.
- JavaScript must be modular and clean up listeners, timers, observers, and pending async work.
- Asset filenames and DOM metadata must not leak secrets, answers, private state, or implementation hints for security-sensitive extensions.
- Third-party assets must keep their license, notice, attribution, and source metadata with the extension.
- Curated asset indexes must stay synchronized with actual files, declared counts, categories, families, checksums, and license metadata.

## Templates

`templates/` contains extension Twig templates.

Rules:

- Keep templates small and extension-scoped.
- Use lowercase, hyphenated filenames.
- Use deterministic translation keys for user-facing text.
- Provider templates should live under the path expected by the provider contract.
- Do not expose secrets, internal paths, private state, or answer labels in rendered markup.

Documented template roots:

- `templates/frontend/**` is referenced as `@frontend/...`.
- `templates/backend/**` is referenced as `@backend/...`.
- `templates/provider/{provider}/**` is referenced through matching provider slots.
- `templates/macros/{extension-slug}/**` may contain extension-owned macros.

## Translations

`languages/{locale}/` contains extension translation catalogues.

Rules:

- Keep every supported locale synchronized.
- Use `ext.{extension-slug}.` as the key prefix.
- Translate labels, buttons, links, placeholders, help text, validation messages, flash messages, empty states, errors, navigation text, and accessibility text.
- Logs, developer exceptions, CLI output, test names, and internal debug strings do not need localization.
- Do not hardcode available languages in extension runtime behavior.

## PHP Source

`src/` contains extension PHP code when the extension needs services, providers, value objects, validators, commands, hooks, or runtime behavior.

Rules:

- Use strict types and PSR-12.
- Prefer Studio and Symfony contracts over custom infrastructure.
- Keep classes small and behavior-focused.
- Validate public input at the boundary.
- Keep extension-owned message codes, settings, service IDs, and cache keys under the extension namespace.
- Avoid relying on undocumented Studio internals. If a host contract is missing, document the assumption and keep the extension-side implementation narrow.

## Tests

`tests/` contains extension-local tests when practical. Some integration tests may belong in the host Studio test suite if they require host extension discovery, service wiring, routes, Live endpoints, or rendered surfaces.

Recommended coverage:

- extension discovery and manifest parsing;
- `extension.php` contribution shape;
- settings defaults and validation;
- provider registration;
- translation key coverage;
- rendered templates or provider output;
- extension lifecycle behavior;
- security-sensitive token, replay, cache, or redaction behavior;
- asset index validation when curated assets are part of the extension.

## Optional Docs

`docs/` may contain extension-specific manuals, design notes, and maintenance files.

Optional files:

- `docs/CLASSMAP.md`: keep current when the extension has public PHP callables, providers, commands, hooks, Twig extensions, event subscribers, or other contributor-facing entry points.
- `docs/WORKLOG.md`: keep current when the extension uses a session worklog workflow.

If these files do not exist, do not create them only to satisfy process. Use README, CHANGELOG, issue notes, or PR notes for durable context instead.

## Security Constraints

Extensions must treat request input and browser-visible state as hostile.

Follow these constraints:

- validate public inputs before using them in paths, selectors, route parameters, cache keys, storage keys, provider keys, commands, subprocesses, or response payloads;
- keep secrets in host configuration or installer-generated storage;
- do not log secrets, private state, sensitive generated output, host paths, environment details, or internal cache keys;
- use opaque identifiers for security-sensitive flows;
- consume one-shot IDs atomically where replay resistance matters;
- set appropriate cache headers for private Live/API responses;
- document cookies, browser storage, roles, ACLs, public endpoints, or retained data;
- provide uninstall, purge, and retention behavior for extension-owned persisted state.

## Verification

Run verification from the Studio project root so extension discovery and the root project rules are active.

Useful focused checks:

```bash
bin/lint extensions/{EXTENSION_SLUG}
php -l extensions/{EXTENSION_SLUG}/src/SomeFile.php
php bin/console lint:container
php bin/phpunit --filter PackageName
php bin/console render:route /some/extension/route
```

Choose checks based on the changed surface:

- metadata/docs: lint Markdown and `.manifest`;
- translations: lint locale catalogues and compare keys;
- Twig: lint templates and render the affected host route or provider surface;
- CSS/JS: lint assets and run JavaScript tests or asset builds where applicable;
- PHP/service changes: syntax, focused tests, and container lint;
- lifecycle/provider changes: host extension discovery/validation once available;
- curated assets: validate index syntax, path existence, declared counts, licenses, duplicates, and ignored-file status.

If a recommended check cannot run because the host contract or command does not exist yet, record that explicitly in PR notes or the final response.

## Release Checklist

Before release or PR readiness:

- `.manifest` matches extension behavior, version, source, scope, and dependencies.
- `extension.php` contributions are extension-scoped and documented.
- README and extension docs describe setup, behavior, constraints, and known host contracts.
- Translation catalogues are synchronized.
- License and third-party attribution files are present.
- Extension-owned assets and indexes are synchronized.
- Tests or documented verification cover changed behavior.
- Security/privacy implications are reviewed.
- Install, update, deactivate, purge, and retained-state behavior are understood.
- Skipped checks and remaining follow-ups are recorded.
