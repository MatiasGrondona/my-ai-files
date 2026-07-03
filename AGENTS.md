# AGENTS.md

Global instructions for all AI agents working on this repository or any project that references
this file. Read this entire file before starting any task.

---

## Skills

Before writing any code or making decisions in the following domains, read the corresponding
SKILL.md file. Skills take precedence over general training knowledge.

Skills are organized in three groups based on their source. All paths are relative to the
`.skills/` directory at the repo root (or its submodule path in a project).

### My Skills — `.skills/android/my/`

Custom skills authored for this project. Highest priority — override anything in the other groups.

| Skill | Path | When to use |
|---|---|---|
| `m3-expressive` | `android/my/m3-expressive/SKILL.md` | Any Android UI: M3 Expressive components, theming, shape morphing, spring animations, adaptive layouts, alpha21 API changes |
| `compose-samples` | `android/my/compose-samples/SKILL.md` | Canonical Compose patterns, architecture references, or real-world implementation examples (Reply, Jetcaster, NowInAndroid…) |
| `snippets` | `android/my/snippets/SKILL.md` | Generating idiomatic Android code for any documented API — source of truth for developer.android.com samples |
| `performance-samples` | `android/my/performance-samples/SKILL.md` | Macrobenchmark, Microbenchmark, Baseline Profile generation, JankStats |
| `android-compose-ui` | `android/my/android-compose-ui/SKILL.md` | Composable stability, recomposition, side effects, lazy lists, animations, previews, accessibility, custom modifiers, design system composables |
| `android-data-layer` | `android/my/android-data-layer/SKILL.md` | Data sources, repositories, DTOs, mappers, Room entities, Ktor HttpClient, safe call helpers, token storage, offline-first |
| `android-di-koin` | `android/my/android-di-koin/SKILL.md` | Koin module definitions per layer, ViewModel injection, assembling modules in :app, `koinViewModel()` in composables |
| `android-error-handling` | `android/my/android-error-handling/SKILL.md` | Generic `Result<T, E>` wrapper, `DataError`, `EmptyResult`, map/onSuccess/onFailure helpers — used across all layers, not just data |
| `android-module-structure` | `android/my/android-module-structure/SKILL.md` | Module layout, dependency rules, Gradle convention plugins, version catalogs, deciding where new modules and features live |
| `android-navigation` | `android/my/android-navigation/SKILL.md` | Type-safe Compose Navigation: route objects, feature nav graphs, cross-feature callbacks, wiring in :app. **Use this over `navigation-3` (Google) for all Nav3 work** |
| `android-presentation-mvi` | `android/my/android-presentation-mvi/SKILL.md` | MVI: State/Action/Event, ViewModel, Root/Screen composable split, UI models, `UiText` error mapping, process death with `SavedStateHandle` |
| `android-testing` | `android/my/android-testing/SKILL.md` | ViewModel unit tests with JUnit5, Turbine, AssertK, `UnconfinedTestDispatcher`, fake repositories, `SavedStateHandle`, Compose UI tests |

### Google Android Skills — `.skills/android/google/` (`android/skills`)

Official Google skills. Use for migration and tooling workflows where these are explicitly listed.

| Skill | Path | When to use |
|---|---|---|
| `agp-9-upgrade` | `android/google/agp-9-upgrade/SKILL.md` | Upgrading Android Gradle Plugin to version 9.x |
| `migrate-xml-views-to-jetpack-compose` | `android/google/migrate-xml-views-to-jetpack-compose/SKILL.md` | Migrating View-based XML layouts to Jetpack Compose |
| `navigation-3` | `android/google/navigation-3/SKILL.md` | Navigation 3 migration tooling only. **Defer to `android-navigation` (My Skills) for all Nav3 implementation work** |
| `r8-analyzer` | `android/google/r8-analyzer/SKILL.md` | Auditing and optimizing R8 configuration for build performance |
| `edge-to-edge` | `android/google/edge-to-edge/SKILL.md` | Implementing edge-to-edge layout support |
| `play-billing-library-version-upgrade` | `android/google/play-billing-library-version-upgrade/SKILL.md` | Upgrading Play Billing Library version |
| `adaptive-layout` | `android/google/adaptive-layout/SKILL.md` | Building responsive UIs for phones, tablets, and foldables |
| `compose-styles` | `android/google/compose-styles/SKILL.md` | Integrating the Jetpack Compose Styles API for component theming |
| `perfetto-trace-analyzer` | `android/google/perfetto-trace-analyzer/SKILL.md` | Analyzing Perfetto traces to diagnose latency, memory, or jank |
| `camerax-migration` | `android/google/camerax-migration/SKILL.md` | Migrating from Camera1/Camera2 to CameraX |
| `appfunctions` | `android/google/appfunctions/SKILL.md` | Implementing AppFunctions to expose app workflows to Android AI agents |
| `android-cli` | `android/google/android-cli/SKILL.md` | Using the Android CLI tool for project creation, device management, and skill management |

### Anthropic Skills — `.skills/anthropic/` (`anthropics/skills`)

Official Anthropic skills for general development and document tasks.

| Skill | Path | When to use |
|---|---|---|
| `docx` | `anthropic/docx/SKILL.md` | Creating or editing Word documents (.docx) |
| `pdf` | `anthropic/pdf/SKILL.md` | Creating, filling, merging, or extracting from PDFs |
| `pptx` | `anthropic/pptx/SKILL.md` | Creating or editing PowerPoint presentations (.pptx) |
| `xlsx` | `anthropic/xlsx/SKILL.md` | Creating or editing spreadsheets (.xlsx) |
| `frontend-design` | `anthropic/frontend-design/SKILL.md` | Building web UI with intentional visual design and typography |
| `mcp-builder` | `anthropic/mcp-builder/SKILL.md` | Building MCP servers to integrate external APIs or services |
| `claude-api` | `anthropic/claude-api/SKILL.md` | Integrating the Anthropic API / Claude SDK in any language |
| `webapp-testing` | `anthropic/webapp-testing/SKILL.md` | Writing automated tests for web applications |
| `web-artifacts-builder` | `anthropic/web-artifacts-builder/SKILL.md` | Building complex multi-component HTML/React artifacts |
| `theme-factory` | `anthropic/theme-factory/SKILL.md` | Applying consistent visual themes to artifacts, slides, or docs |
| `canvas-design` | `anthropic/canvas-design/SKILL.md` | Creating visual art, posters, or static design assets |
| `skill-creator` | `anthropic/skill-creator/SKILL.md` | Creating, modifying, or evaluating SKILL.md files |

---

## Changelog

### File: `CHANGELOG.md`

Maintain a `CHANGELOG.md` at the root of every project. Update it at the end of every development
session before closing.

### Version format: `[MAJOR.MINOR.PATCH-TAG]`

```
[0.0.00-alpha]
 │  │   │   └── Tag: alpha → beta → stable (changes only when explicitly instructed)
 │  │   └────── Patch: double-digit, increments every development session
 │  └────────── Minor: increments when a feature is stable enough to begin beta testing
 └───────────── Major: increments on stable release or major version
```

- **Major** (`0`): Changes only when the app reaches stable release or a major version milestone.
- **Minor** (`0`): Changes when a new feature is stable enough for beta; resets patch to `00`.
- **Patch** (`00`): Double-digit. Increments every development session. The only one that changes
  frequently.
- **Tag**: Starts as `alpha`. Does not change unless explicitly instructed. Once changed, stays
  on the new tag until explicitly instructed to change again.

### Changelog template

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [0.0.00-alpha]

### BUG Fixes
- Description of the fix on this version

### DESIGN Fixes
- Description of the design change on this version

### PERFORMANCE Fixes
- Description of the performance improvement or fix on this version.
```

---

## TODO

### File: `TODO.md`

Maintain a `TODO.md` at the root of every project. Update it whenever tasks are added, completed,
or change status.

### Structure

Two top-level sections — `PENDING` and `COMPLETED` — each containing the same four categories:

- **PROJECTS TODOs**: General project tasks — setup, dependencies, agent configuration,
  project-level decisions.
- **BUG Fixes**: Defects, crashes, incorrect behavior.
- **DESIGN Fixes**: Visual, UX, or layout corrections.
- **PERFORMANCE Fixes**: Speed, memory, frame rate, or benchmark improvements.

Move tasks from `PENDING` to `COMPLETED` (changing `- [ ]` to `- [x]`) when done. Never delete
completed tasks.

### TODO template

```markdown
# TODO


## PENDING

### PROJECTS TODOs
- [ ] task

### BUG Fixes
- [ ] task

### DESIGN Fixes
- [ ] task

### PERFORMANCE Fixes
- [ ] task

## COMPLETED

### PROJECTS TODOs
- [x] completed task

### BUG Fixes
- [x] completed task

### DESIGN Fixes
- [x] completed task

### PERFORMANCE Fixes
- [x] completed task
```

---

## Documentation

Write inline documentation for all public functions, classes, and methods. Follow the standard
format for the language in use:

- **Kotlin**: KDoc (`/** */`) with `@param`, `@return`, `@throws`
- **Java**: Javadoc (`/** */`) with `@param`, `@return`, `@throws`
- **TypeScript / JavaScript**: JSDoc (`/** */`) with `@param`, `@returns`, `@throws`
- **HTML**: Comment blocks (`<!-- -->`) for non-obvious structure decisions

Each doc block must include:
1. A one-sentence summary of what the function does.
2. Each parameter: name, type, and purpose.
3. Return value: type and meaning (omit for `Unit`/`void`).
4. Any exceptions thrown, if relevant.

```kotlin
// Kotlin example
/**
 * Loads the user profile from the repository and updates the UI state.
 *
 * @param userId The unique identifier of the user to load.
 * @return A [Flow] emitting [UserUiState] reflecting the latest profile data.
 * @throws UserNotFoundException if no user exists for the given [userId].
 */
fun loadUser(userId: String): Flow<UserUiState>
```

---

## Version Control

- After completing a task (feature, fix, or refactor), commit the changes with a conventional commit message (`feat:`, `fix:`, `refactor:`, `chore:`, etc.) summarizing what was done.
- Do not commit partial or broken work — only commit when the task compiles/builds successfully.
- Do not push to the remote repository automatically. Only push when explicitly instructed with a command like "push" or "push to remote".
- When pushing, push to the current branch's upstream unless told otherwise.
- Never force-push unless explicitly instructed.

---

## Conventions

### Android

- Always follow **Material 3 Expressive** for all UI. Read `.skills/android/my/m3-expressive/SKILL.md` before writing any UI code.
- Use Jetpack Compose for all new UI. Do not introduce new View-based UI unless explicitly
  instructed.

### Web

- Default framework: **Angular**. Use another framework only when explicitly instructed.
- For simple web applications: prefer a **single HTML file** unless the scope clearly requires
  a multi-file setup. When unsure, ask before creating multiple files.
- Even in a single HTML file, always write **TypeScript** or follow TypeScript conventions
  (types, interfaces, explicit return types). If the environment does not support TypeScript
  compilation, apply the conventions in JavaScript as strictly as possible.

### Package Manager

- Always use **pnpm**. Never use npm or yarn.

---

## Build Output Naming

Whenever the version is updated in `CHANGELOG.md` or in the app's `versionName`, also update
the APK output file name to match. Keep these three in sync at all times:

- `CHANGELOG.md` — latest version entry (e.g. `[0.1.03-alpha]`)
- `versionName` in `app/build.gradle.kts`
- `archivesName` in `app/build.gradle.kts`

### Implementation

```kotlin
// app/build.gradle.kts
android {
    defaultConfig {
        versionName = "0.1.03-alpha"
    }
    archivesName = "swipe-${defaultConfig.versionName}"
}
```

This produces `swipe-0.1.03-alpha-debug.apk` / `swipe-0.1.03-alpha-release.apk`
instead of `app-debug.apk`.

> **AGP version note:** This project uses AGP 9.x. Use `archivesName` — `archivesBaseName`
> (the AGP 8.x property) is deprecated and will not work.

### What to update every session

1. Bump `versionName` in `app/build.gradle.kts`
2. Add the matching version entry to `CHANGELOG.md`
3. Verify `archivesName` reflects the new version — it uses `defaultConfig.versionName`
   directly so it updates automatically if set as shown above

---

## What Not To Do

- **Do not use `npm`** — use `pnpm` for all package installation and script execution. `npm` is
  prohibited for security reasons.
- **Do not create new View-based Android UI** unless explicitly instructed.
- **Do not change the version tag** (`alpha`/`beta`/`stable`) unless explicitly instructed.
- **Do not skip reading the relevant SKILL.md** before working in a domain covered by a skill.
