# Repository Agent Guide

> **Status**: Active  
> **Updated**: 2026-06-19  
> **Owner**: Dominik Letica, OpenAI/Codex  
> **Purpose:** Provide practical, repository-specific instructions and binding project rules for coding agents working on Studio packages.  

## Binding Rule Source
- `AGENTS.md` is the binding repository-wide rule source for architecture, naming, process, security, localization, and audit decisions.
- This file mirrors the Studio core agent guide where package repositories need the same standards, and only removes or adapts rules that are specific to the core application.
- Update `AGENTS.md` directly when package architecture, naming, process, security, localization, audit, or verification rules change.

## Project Rules

### Pre-1.0 Development
- No stable production compatibility contract must be guaranteed before the first stable package `1.0.0` release.
- Before `1.0.0`, prefer clean design over compatibility shims, data migrations, or legacy behavior.
- Remove obsolete package code paths instead of preserving backward compatibility unless the user explicitly asks otherwise.
- If the package introduces persisted data before `1.0.0`, keep schema, migrations, tests, docs, package metadata, and host integration assumptions aligned in the same change.
- If a host contract is still in flux before `1.0.0`, document the assumption and keep the package implementation narrow enough to adapt later.

### Package Identity
- `.manifest` is the package identity source and must stay aligned with package behavior.
- Keep `PACKAGE_SLUG`, `PACKAGE_NAME`, `PACKAGE_DESCRIPTION`, `PACKAGE_VERSION`, `PACKAGE_SCOPE`, `PACKAGE_DEPENDENCIES`, `PACKAGE_LICENSE`, `PACKAGE_HOMEPAGE`, `PACKAGE_CHANNEL`, `PACKAGE_SOURCE`, and `PACKAGE_DATE` current.
- The package slug is the owner namespace for package-owned assets, templates, translations, settings, live endpoints, provider keys, message keys, CSS selectors, storage keys, cache keys, and generated files.
- Do not use `system`, `studio`, or another package slug for new package-owned identifiers.
- Package contributions must remain installable, inspectable, disableable, and removable through Studio's package lifecycle.
- Studio core must not depend on files under this repository; this repository may depend only on documented or deliberately accepted host contracts.

### Database Support
- Packages should avoid introducing database requirements unless the package feature genuinely needs persisted state.
- When package persistence is needed, follow the host application's MariaDB/MySQL, SQLite, and PostgreSQL portability rules where practical.
- Prefer portable Doctrine types, portable indexes, explicit columns for frequently filtered values, and app-level validation over vendor-specific SQL behavior.
- JSON columns are acceptable for flexible package configuration, metadata, and content-like payloads when core read paths do not rely on vendor-specific JSON operators.
- Do not require database-specific features unless the package explicitly declares and documents a minimum database/version requirement.
- Package-owned persisted state must have a clear install, update, deactivate, purge, and retention story.

### Content Revisions
- Packages that contribute import, editor, preview, review, revert, or content-retention behavior must respect Studio's content revision model.
- Imports may stage proposed changes as new revisions and activate them only after review when the host contract supports that workflow.
- Cleanup and retention should be driven by nullable active pointers and configuration, not by hard-deleting historical rows by default.
- Packages that do not touch content revision behavior do not need to invent a revision layer.

### Architecture And Development Rules
- Modularity is preferred over monolithic implementations. Split large classes, controllers, services, tests, templates, JavaScript modules, and helpers when the extracted boundary improves responsibility, reuse, readability, or LLM context stability.
- Files should stay below roughly 300 lines where practical because this size remains reliable for agent context handling, review, and patching. Treat this as a soft limit: split files when a clear responsibility boundary exists, but do not fragment a naturally cohesive file only to satisfy the line target.
- Public and contributor-facing callable, interface, hook, event, command, route, payload, translation-key, setting, provider-key, and extension-point names must be clear, consistent, and easy to document.
- New or refactored package-owned technical identifiers must follow one stable owner/scope/name convention. The package owner is the package slug from `.manifest`; reserve `system` for core-owned behavior and `Studio`/`studio` for branding or deliberately retained host surfaces.
- Package UI, template, and CSS names should stay package-scoped. Use `{package-slug}-*` for owner-level selectors and `{package-slug}-{provider-scope}-*` for provider selectors. Package-owned selectors must remain under the package slug and declared scope so package validation can detect collisions.
- Prefer plain domain names when a storage boundary is already package-owned and no package/user collision is possible. Use the package owner prefix where it protects a shared namespace such as cookies, browser storage, session attributes, service tags, package identities, generated assets, live resources, API resources, or package-facing extension points.
- CLI command names should prefer Symfony-style domain namespaces over product branding. A product/package prefix is acceptable when it avoids a real namespace collision or declares package ownership.
- Package logs, if any, should stay descriptive and environment-scoped. Avoid adding product branding to every log filename unless the log lives in a shared host namespace and needs collision protection.
- Runtime errors, validation failures, operational diagnostics, and user-facing feedback should use the host shared `Message`, `MessageCode`, `MessageKey`, `WorkflowResult`, or `MessageException` layer wherever practical. Hard exceptions with literal text are allowed only for consciously chosen low-level invariants or unrecoverable adapter failures.
- Package-owned message code/key constants must live in package-owned catalogue classes close to the owning namespace when the package contains PHP code. Values must stay in the package-owned machine namespace and must not override `system`, core, or other package namespaces; system catalogues win conflicts.
- Public hook descriptors must be registered close to the event owner. Core registries aggregate providers for documentation, tooling, debug output, and package API validation; package code must not turn them into central hand-built lists.
- Available languages must be discovered from translation catalogues, runtime configuration, or explicit content data. Do not hardcode language variants in runtime control flow, default API parameters, or administrative form fields.
- Core/system cookies remain first-party and technically scoped. Packages that introduce cookies, browser storage, analytics, advertising, or tracking must own their consent requirements, documentation, and isolation from core technical cookies.
- Child processes and detached runners must use host process/environment helpers unless a direct process call is intentionally local and documented. Application subprocesses must inherit Symfony Dotenv-derived configuration and must filter web/CGI request context before execution.
- Add small wrapper or helper APIs when they make public or extension-facing behavior easier to explain, safer to call, or less error-prone.
- Prefer Symfony, Doctrine, Twig, Messenger, Validator, Serializer, Process, Filesystem, Security, EventDispatcher, Form, Translation, AssetMapper, Stimulus, and other maintained vendor capabilities over custom infrastructure unless the custom abstraction has clear package-specific value.
- Additional vendor packages are acceptable when they reduce custom maintenance, improve portability/security, or integrate cleanly with Studio and Symfony without making the package unnecessarily heavy.
- Keep tests behavior-focused. Secure public behavior, cross-platform assumptions, security boundaries, package lifecycle behavior, and data-model guarantees without pinning fragile template, CSS, or implementation details.
- Performance and data-model decisions must be justified by expected behavior and scale, including identifier strategy, indexes, pagination, filtering, sorting, caching, filesystem scans, process spawning, request/visitor identifiers, media processing, generated assets, and full-tree work.
- Security and misuse resistance must be considered for public entry points, sessions, tokens, visitor/request identity, subprocesses, filesystem access, package/module boundaries, logging, audit data, secrets, browser-visible metadata, and environment propagation.
- Feature drafts, existing implementation choices, and early pre-`1.0.0` assumptions are guidance, not law. Prefer a simpler, safer, more Symfony-native, or more maintainable design when evidence supports changing course.

### Package Assets
- Every used asset path must resolve to an existing package-owned file.
- Keep third-party `LICENSE`, `NOTICE`, attribution, or source files next to the curated assets they cover unless the package documents a clearer license location.
- Keep private source assets private to the package when runtime behavior depends on opacity, anti-abuse properties, licensed source material, or server-side transformation.
- Do not expose semantic internal asset names, source filenames, answer labels, or implementation hints to browsers when those details weaken security or product behavior.
- Avoid visually tiny, ambiguous, copyrighted, trademark-sensitive, or hard-to-distinguish assets in default curated sets unless the package explicitly needs them and documents the trade-off.

### Architecture And Drift Audits
- Run broad architecture and project-rules drift audits as reusable review gates.
- Use audits to verify that current package code and new feature work still follow the architecture and development rules above.
- Critically review feature drafts, existing implementation choices, and early pre-`1.0.0` assumptions during audits instead of treating them as binding.
- Review performance and data-model decisions critically, including UUIDs versus auto-increment identifiers, indexes, pagination, filtering, sorting, caching, filesystem scans, process spawning, request/visitor identifiers, media processing, generated assets, and full-tree work.
- Review security and misuse resistance around public entry points, sessions, tokens, visitor/request identity, subprocesses, filesystem access, package/module boundaries, logging, audit data, secrets, browser-visible metadata, and environment propagation.
- Capture audit findings with evidence, impact, recommendation, and priority. Apply small safe improvements directly; split larger refactors into dedicated follow-up issues or audit PR slices.

## Operating Principles
- Read the existing package files, `.manifest`, documentation, and relevant host contracts before changing behavior. Prefer local patterns over new abstractions.
- Make use of Studio and Symfony native packages and features whenever applicable and keep the package as lightweight, compatible, and clean as possible.
- Focus on modular implementations and keep file sizes small for optimized context handling and readability.
- Keep changes focused on the user request. Do not refactor unrelated code unless it is required to complete the task safely.
- Preserve user or collaborator changes. Never revert files you did not intentionally change unless the user explicitly asks for it.
- Repository text must be English, including code comments, documentation, commit messages, UI copy source strings, and changelog entries.
- When instructions conflict, follow the user's explicit request for the current task and call out any repository-rule trade-off.
- Prefer graceful production flows: handle recoverable errors, log useful context, and avoid throwing where a user-facing recovery path is possible.

## Project Map
- `.manifest` contains package metadata consumed by Studio.
- `assets/` contains package frontend sources that Studio may mirror, import, or bundle.
- `templates/` contains package Twig views. Provider templates belong under `templates/provider/{provider-scope}/...`.
- `languages/{locale}/` contains package translation source catalogues.
- Package-specific asset directories such as `fixtures/`, `icons/`, `media/`, `schemas/`, or other package-owned data folders may contain curated data and indexes when the package needs them.
- `docs/` contains package user or developer documentation when present.
- `README.md`, `CHANGELOG.md`, `SECURITY.md`, `LICENSE`, and `.github/` contain package-facing repository documentation and contribution metadata.
- Tests, PHP sources, package configuration, or build scripts may be added when the package grows; keep them package-scoped and documented here when they become part of the repository contract.
- `.codex/` may contain agent-only notes, helper scripts, context cache, and environment/tooling notes. Production code, tests, build scripts, and release workflows must not depend on files under `.codex/`.

## Before Editing
- Check the package rules in this file, `.manifest`, README, CHANGELOG, SECURITY, and relevant package docs for active TODOs and recent context before code changes.
- Check the host Studio `AGENTS.md`, package contracts, and framework/version recap when behavior depends on precise current host APIs.
- Prefer reusable repository or host helpers over ad-hoc command snippets for agent-only development tasks. Promote helpers into package or host tooling only when they become useful to other developers or required by tests, CI, build, setup, or production workflows.
- For behavior changes, identify the matching documentation and test locations before editing.
- For UI or rendered-output changes, identify affected Twig templates, translations, assets, provider surfaces, and host routes.
- For indexed asset changes, inspect the index file, actual asset files, license files, and selection policy or metadata together.

## Change Expectations
- Behavior changes must include matching tests, documentation updates, changelog notes, and package metadata/index updates when relevant.
- Documentation-only changes should follow the host documentation style where practical; tests are not required unless examples or tooling behavior change.
- Translation changes must keep matching source catalogue files and keys synchronized across all locale directories under `languages/`.
- Refactors before the first public package `1.0.0` release may remove obsolete code instead of keeping compatibility shims, but callers, tests, docs, translations, package metadata, and examples must be updated immediately.
- Asset-index changes must keep `itemsCount`, source paths, license references, category/family constraints, and actual files synchronized.
- If a requested narrow change exposes unrelated drift, fix it only when it blocks the task; otherwise record the follow-up in CHANGELOG, package docs, issue notes, or final notes as appropriate.

### Review Finding Fixes
- Before applying a fix for a review finding, trace the affected boundary from source to sink and inspect adjacent, related, and analogous code paths that share the same classifier, subscriber, guard, resolver, route family, subject selection, response behavior, storage boundary, or policy decision.
- Prefer fixing the narrowest central boundary that covers all affected paths. Apply a path-local fix only when evidence shows the issue is truly path-specific.
- Keep review fixes simple, modular, and minimally invasive. Do not broaden them into unrelated refactors, compatibility shims, or speculative redesigns.
- While tracing the affected boundary, actively look for additional unreported edge cases, including bypasses, abuse paths, privacy leaks, performance regressions, setup/pre-auth behavior, disabled-feature fallbacks, response redaction, cache/storage failure behavior, package lifecycle behavior, and asset exposure.
- Fix small in-scope adjacent issues directly when they share the same boundary and risk profile. Record larger or behavior-changing follow-ups in package docs, issue notes, or final notes instead of hiding them inside the review fix.
- Add or update regression coverage for the reported finding and any adjacent paths changed by the fix. When an analogous path is inspected and intentionally not changed, make that reasoning clear in the final notes or PR response where useful.

### PR Readiness Audits
- Before marking a branch, pull request, or feature slice ready for review, run the PR-readiness checklist as a real audit pass over the branch diff and the affected runtime surfaces. Do not treat checklist items as passive boxes to tick.
- The audit must explicitly review security/privacy considerations; public entry points; authentication, authorization, sessions, secrets, browser storage, cache, and response redaction; package/module boundaries; access levels; route/API/live endpoint scopes; naming and collision risks; setup/init/CI behavior; cross-platform behavior; disabled-feature fallbacks; process and environment handling; package default coverage; translations and user-facing copy; license files; asset index consistency; project-rule, architecture, naming, documentation, and performance drift; and captured follow-up tasks.
- Use evidence from code inspection, focused tests, render checks, linting, documentation diffs, package metadata diffs, and default/config coverage as appropriate for the changed surface. If a checklist item is not applicable, record why instead of silently skipping it.
- Fix small readiness issues directly when they are in scope and low risk. Record larger, behavior-changing, or host-contract follow-ups clearly instead of hiding them inside unrelated package work.
- PR notes must summarize the readiness audit outcome, including verification commands, skipped checks or proof gaps, documentation/changelog status, translation status, security/privacy considerations, license status, and remaining follow-ups.

## Build and Verification Commands
- Use the host Studio repository to validate package behavior whenever the changed surface depends on Studio package discovery, provider registration, Twig rendering, translations, Symfony services, Live/API endpoints, or asset processing.
- `bin/lint <package-paths...>` from the host repository is the preferred focused lint entry point for Markdown, JSON, YAML, CSS, JavaScript, Twig, PHP syntax, local icon references, and Git whitespace checks.
- `php -l <path>` checks PHP syntax for changed PHP files when this package contains PHP code.
- `php bin/console lint:container` in the host repository validates Symfony container wiring after package service or configuration changes.
- `php bin/console tailwind:build` in the host repository validates Tailwind/CSS integration when package CSS affects the built application.
- `php bin/console asset-map:compile` is production/release-only in the host application. Do not run it for local development or normal verification; if production output is created locally, remove that generated output from the worktree.
- `php bin/console ux:icons:lock` imports referenced Symfony UX/Iconify icons into the host application when needed. Avoid bulk-locking complete icon sets without a concrete need.
- `php bin/phpunit` or focused PHPUnit tests in the host repository should cover package lifecycle, provider registration, live endpoint, token validation, rendered output, and package-specific behavior when those surfaces exist.
- `bin/jstest` or focused JavaScript tests should cover DOM-free package JavaScript behavior when available.
- For package-only edits, run syntax checks appropriate to changed files, including JSON checks for package indexes, YAML checks for `languages/**/*.yaml`, CSS checks for `assets/**/*.css`, JavaScript checks for `assets/**/*.js`, and Markdown parse checks for docs.
- Before committing, use the host `bin/lint --diff` or the relevant focused lint command for Git-aware whitespace checks. Markdown files may contain intentional two-space hard line breaks; preserve those hard breaks when reviewing whitespace output from raw Git commands.
- `php bin/console render:route /<route>` in the host repository renders a route for Twig, translation, and debug user/role review when a package surface is reachable through a host route.

## Verification Matrix
- PHP-only logic: run targeted PHPUnit coverage and `php -l` for edited PHP files.
- Service, DI, security, package registration, or configuration changes: run targeted tests and host `php bin/console lint:container`.
- Twig, translation, or UX copy changes: run host `bin/lint <changed translation/template paths...>` and render affected routes or provider surfaces when available.
- Asset or Stimulus changes: prefer host `bin/lint <changed path...>` for focused JavaScript, JSON, CSS, YAML, Twig, Markdown, and PHP syntax checks, run `bin/jstest` or focused JavaScript tests when DOM-free behavior can be covered, then run the relevant development asset build command and targeted UI/functional checks when build output or rendering can change.
- Indexed asset changes: validate syntax, asset existence, declared counts, license files, category/family or selection policy, duplicate IDs, ignored-file status, and absence of unsafe client-visible semantic leakage.
- Security-sensitive rendering or generated response changes: verify generated output is valid and nonblank where applicable, secrets or answers are not present in client-visible metadata, one-shot IDs are consumed when used, replay attempts fail when relevant, and cache headers match the sensitivity of the response.
- Documentation changes: run a Markdown parse/lint check, then verify style, relative links, package metadata alignment, and current behavior.
- If a recommended verification step cannot run, record the reason in the final response and, when relevant, in package docs, issue notes, or PR notes.

## Coding Style
- Respect `.editorconfig` for whitespace and indentation rules; Markdown may intentionally disable automatic trailing-whitespace trimming because two trailing spaces are meaningful hard line breaks.
- PHP follows PSR-12 with four-space indentation and `declare(strict_types=1);` where applicable.
- YAML keys use `snake_case`.
- Twig templates use lowercase, hyphenated filenames such as `field.html.twig`.
- Stimulus controllers use `snake_controller.js` names and register through the Symfony loader when the host integration uses Stimulus.
- JavaScript files should be modular, deterministic, and should clean up listeners, timers, observers, and pending async work when disconnected or replaced.
- CSS should stay package-scoped and avoid global resets, broad element selectors, and unowned custom properties.
- Use automated formatters when available, such as `php-cs-fixer`; otherwise keep diffs manually PSR-12-compliant.
- Keep comments succinct and useful. Add comments only when they clarify non-obvious intent.
- Do not commit secrets, local credentials, generated caches, build output, vendor directories, or host runtime files.

## Translations
- Every user-facing string in Twig, PHP, or JavaScript must use a deterministic translation key.
- Package translation keys follow `pkg.{package-slug}.section.token`, for example `pkg.example-package.placeholder`.
- Keep matching translation source catalogue files and keys in sync across all locale directories under `languages/`, with English used as the comparison reference when available.
- User-facing strings include labels, buttons, links, placeholders, help text, validation messages, flash messages, empty states, error pages, navigation text, and accessibility text.
- Logs, developer exceptions, CLI output, test names, and internal debug strings do not need localization.
- Do not hardcode available languages in runtime package behavior; derive them from the host runtime, translation catalogues, or explicit package configuration.
- For rendered Twig review, use host `php bin/console render:route /<route>` when available and then host `bin/lint <changed translation/template paths...>`.

## Documentation
- Follow the host `dev/STYLEGUIDE.md` for Markdown documentation where practical.
- Update `README.md` when package setup, installation, public behavior, settings, compatibility, assets, or host integration changes.
- Update `CHANGELOG.md` for meaningful package changes.
- Update `SECURITY.md` links and package-specific reporting instructions when the repository URL, support policy, or security model changes.
- Update `docs/` when user-facing or developer-facing package behavior needs more detail than the README should carry.
- Keep examples aligned with `.manifest`, package paths, translation keys, asset layout, and current host contracts.
- Documentation-only changes should still be checked for Markdown syntax and link accuracy.
- Security-sensitive behavior should be documented without exposing exploit-enabling internals.
- Read relevant Markdown files end-to-end before editing them. Full documentation sweeps are required for release or explicit documentation-review tasks, not for every small code change.

## Tests
- Use PHPUnit with namespaces mirroring production code when the package contains PHP code.
- Test method names should be clear, using `testSomething()` or `it_should_doSomething()`.
- Keep tests deterministic: seed random choices, freeze time where practical, avoid external services, and clean up temporary files, database state, and cache state.
- Add or adjust coverage for every behavior change. Do not skip tests for new logic simply because the package is pre-`1.0.0`.
- Prefer targeted tests while developing, then run broader host or package suites before PRs or high-risk changes.
- Add regression coverage for security boundaries, package lifecycle behavior, asset index validation, token validation, replay prevention, translation coverage, and rendered provider output when those surfaces change.
- When host integration is required and no package-local harness exists, place tests in the host Studio repository or document the missing host test hook clearly.

## Class Map
- If `docs/CLASSMAP.md` exists in this package, keep it current when adding, removing, or changing relevant callables.
- Relevant callables include controller actions, console commands, services with public behavior, event listeners/subscribers, form types, Twig extensions/components, security voters, package providers, live components, hooks, and other entry points contributors need to find quickly.
- Include or update related test references where available.
- If no package-local `docs/CLASSMAP.md` exists, do not create one only to satisfy this guide unless the package has enough public surface to justify it.

## Worklog
- If `docs/WORKLOG.md` exists in this package, record meaningful code, behavior, documentation, and tooling changes there.
- Keep active worklog entries session-based with branch context when the package uses that workflow.
- Note completed work, verification performed, and TODOs or follow-ups that remain.
- If no package-local `docs/WORKLOG.md` exists, use `CHANGELOG.md`, issue/PR notes, or final notes for durable follow-up context as appropriate.
- Do not use the worklog or changelog as a substitute for fixing issues that are part of the current task.

## Security and Configuration
- Public package code must assume the browser and request input are hostile.
- Keep secrets in host configuration or installer-generated storage, never committed package files.
- Do not log secrets, private runtime state, sensitive generated output, package install paths, host environment details, internal cache keys, or user-provided sensitive data.
- Validate public package inputs at the boundary and normalize before using them in filesystem paths, cache keys, selectors, provider keys, route parameters, commands, subprocesses, or response payloads.
- Never trust package asset filenames, request-provided provider keys, package IDs, tokens, or resource identifiers without package-owned or host-owned validation.
- Review response redaction for admin, API, live, logs, debug payloads, and exception pages.
- Review rate-limit, replay, session fixation, copied-session, cross-tab, disabled-feature, cache-failure, and uninstall/purge behavior for public package endpoints and stateful workflows.
- Security-sensitive Live/API responses should use appropriate cache headers and must not expose secrets or internal state.
- Captcha, invite, reset, webhook, payment, authentication, authorization, or other security-sensitive packages must validate server-side, use opaque identifiers, consume one-shot IDs atomically where relevant, and verify replay/failure behavior.
- Consider accessibility and recovery flows when package behavior can block legitimate users.
- Document role, ACL, provider, cookie/storage, or public endpoint changes in the relevant package or host manual.
- Do not expose secrets in committed configuration, docs, logs, fixtures, screenshots, or test artifacts.

## Review Mode
- In code review, lead with findings ordered by severity and include file and line references.
- Review-fix implementation must follow the Review Finding Fixes rules under Change Expectations before applying code changes.
- PR-readiness sign-off must follow the PR Readiness Audits rules under Change Expectations instead of only copying checklist items.
- Verify changelog, documentation, tests, package metadata, optional docs/CLASSMAP, optional docs/WORKLOG, asset indexes, translations, screenshots, license files, security notes, and PR checklist items when they are relevant to the reviewed change.
- Check drift between code and package docs or host contracts; update it only when asked to make changes, otherwise report the drift.
- Review translation coverage with host `bin/lint <changed translation paths...>` when user-facing copy changed.
- Review relevant Markdown files for completeness and link health. Run or schedule link checks when possible.

## Commit and Pull Request Guidance
- Use present-tense imperative commit messages scoped to one logical change, for example `Add package asset index`.
- Reference issues with `[#123]` or GitHub keywords when applicable.
- Pull requests must include a change summary, testing notes, documentation notes, and any security, license, package lifecycle, or migration considerations.
- Keep PRs focused and record deferred follow-up work in issue/PR notes, package docs, or CHANGELOG as appropriate.

## References
- Package metadata: `.manifest`
- README entry point: `README.md`
- Package changelog: `CHANGELOG.md`
- Security policy: `SECURITY.md`
- Package translations: `languages/**`
- Package templates: `templates/**`
- Package frontend assets: `assets/**`
- Optional package class map: `docs/CLASSMAP.md`
- Optional package worklog: `docs/WORKLOG.md`
- Package-specific asset indexes: package-owned asset directories such as `fixtures/**`, `icons/**`, `media/**`, `schemas/**`, or other curated data folders when present
- Host Studio repository: `/Volumes/Projekte/studio`
- Host repository guide: `/Volumes/Projekte/studio/AGENTS.md`
- Host documentation style guide: `/Volumes/Projekte/studio/dev/STYLEGUIDE.md`
