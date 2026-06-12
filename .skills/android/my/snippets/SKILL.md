---
name: snippets
description: >
  Reference skill for the official android/snippets repository — the source of truth for all
  code samples embedded in developer.android.com guides and API documentation. Use when
  generating or verifying idiomatic Android code for any documented API: Jetpack Compose UI,
  animations, state, modifiers, text, lists, graphics, accessibility, interop, theming, DataStore,
  Gradle/KTS build scripts, Kotlin patterns, Bluetooth LE, Play Billing, Wear OS, Android XR,
  TV, KMP, or Views. Trigger this skill any time the user asks for "the correct way to do X in
  Android", references a developer.android.com guide, or needs a snippet that matches what the
  official docs show — even if they don't explicitly name the snippets repo.
sources:
  - https://github.com/android/snippets
metadata:
  author: Matias
  last-updated: '2026-06-10'
  keywords:
    - android-snippets
    - developer-android-com
    - jetpack-compose
    - official-samples
    - datastore
    - gradle-kts
    - kotlin
    - wear-os
    - android-xr
    - bluetooth-le
    - play-billing
    - kmp
    - views-interop
    - accessibility
    - animations
    - state
    - modifiers
    - lazy-lists
---

## Overview

`android/snippets` is the **single source of truth** for all code samples that appear in
developer.android.com guides and API reference pages. Every snippet in the repo is compiled and
tested against real Android SDK versions — if it's in this repo, it is the officially sanctioned
way to write that code.

The repo is organized by technology domain at the top level, each domain being an independent
Android/Kotlin project. Within the Compose domain, snippets are organized by topic into packages
that directly mirror the sections of the Compose documentation.

> **Key insight for code generation:** When generating Android code for a documented API, this
> repo contains the *exact* snippet that the docs page embeds. Always prefer patterns from here
> over general Kotlin/Compose knowledge.

---

## 1. Repo Structure

```
android/snippets/
  compose/          ← Jetpack Compose (largest domain, see §2)
  kotlin/           ← Kotlin language features used in Android context
  datastore/        ← Proto DataStore and Preferences DataStore
  gradle/           ← Gradle KTS build scripts, version catalogs, convention plugins
  kmp/              ← Kotlin Multiplatform (shared logic, expect/actual)
  views/            ← Traditional View-based UI patterns
  bluetoothle/      ← Bluetooth Low Energy scanning, connecting, GATT
  playbilling/      ← Google Play Billing Library integration
  tv/               ← Android TV / Leanback with Compose
  wear/             ← Wear OS with Compose
  wearcompanion/    ← Companion app pairing for Wear OS
  watchface/        ← Watch Face Format and WFF XML
  watchfacepush/    ← Watch Face Push API (validator)
  xr/               ← Android XR (spatial computing)
  identity/
    credentialmanager/ ← Credential Manager, passkeys, sign-in
  misc/             ← One-off snippets that don't fit other domains
  buildscripts/     ← Shared Gradle build configuration
  spotless/         ← Code formatting configuration (Spotless plugin)
```

---

## 2. Compose Snippet Packages

All Compose snippets live under:
`compose/snippets/src/main/java/com/example/compose/snippets/`

Each sub-package maps 1:1 to a section of the Compose documentation:

| Package | Documentation section | Key files |
|---|---|---|
| `accessibility/` | Accessibility in Compose | `AccessibilitySnippets.kt` |
| `animations/` | Animation in Compose | `AnimationSnippets.kt`, `AnimatedContentSnippets.kt`, `SharedTransitionSnippets.kt` |
| `components/` | M3 components (Button, TextField, TopAppBar, Navigation…) | `ButtonSnippets.kt`, `TextFieldSnippets.kt`, `Navigation.kt` |
| `graphics/` | Graphics in Compose (Canvas, shaders, brushes) | `GraphicsModifiersSnippets.kt`, `BrushSnippets.kt` |
| `images/` | Images and icons | `ImageSnippets.kt` |
| `interop/` | Compose + Views interoperability | `InteroperabilityAPIsSnippets.kt` |
| `layouts/` | Layouts (Box, Column, Row, ConstraintLayout, custom) | `LayoutSnippets.kt` |
| `lists/` | Lazy lists and grids | `LazyListSnippets.kt` |
| `modifiers/` | Modifier system, custom modifiers, ordering | `ModifierSnippets.kt` |
| `navigation/` | Navigation in Compose | `NavigationSnippets.kt` |
| `resources/` | Resources (strings, drawables, fonts) | `ResourcesSnippets.kt` |
| `state/` | State, `remember`, `CompositionLocal`, side effects | `StateSnippets.kt`, `CompositionLocalSnippets.kt` |
| `text/` | Text, typography, `AnnotatedString` | `TextSnippets.kt` |
| `theming/` | Material theming, color schemes, dark mode | `DarkThemeSnippets.kt`, `ThemingSnippets.kt` |
| `tooling/` | `@Preview`, `PreviewParameter`, `LocalInspectionMode` | `AndroidStudioComposeSnippets.kt` |

---

## 3. Snippet Tagging Convention

Every snippet in the repo is wrapped in `[START]` / `[END]` comment tags. These tags are what the
documentation site uses to embed the snippet inline in the guide page.

```kotlin
// [START android_compose_layouts_column_simple]
@Composable
fun ArtistCard(artist: Artist) {
    Column {
        Text(artist.name)
        Text(artist.lastSeenOnline)
    }
}
// [END android_compose_layouts_column_simple]
```

**Tag naming pattern:** `android_<domain>_<topic>_<descriptor>`

Examples:
- `android_compose_animations_animated_content`
- `android_compose_state_remember_saveable`
- `android_compose_tooling_simple_composable_preview`
- `android_compose_lists_lazy_column_simple`

> **Implication:** When a developer.android.com guide shows a code sample, that exact code (with
> those exact imports and that exact structure) lives in this repo under the matching tag. If you
> need to reproduce what a specific doc page shows, search for the tag name — it directly
> corresponds to the section URL path.

---

## 4. Key Patterns by Domain

### 4.1 Compose — State & Side Effects
```kotlin
// [START android_compose_state_remember_mutable_state_of]
var count by remember { mutableStateOf(0) }
// [END android_compose_state_remember_mutable_state_of]

// rememberSaveable — survives process death and config changes
var selectedTab by rememberSaveable { mutableStateOf(0) }

// LaunchedEffect — coroutine tied to composable lifecycle
LaunchedEffect(key1 = userId) {
    viewModel.loadUser(userId)
}

// CompositionLocal usage
val contentColor = LocalContentColor.current
```

### 4.2 Compose — Lazy Lists
```kotlin
// LazyColumn with keys for stable identity
LazyColumn {
    items(items = messages, key = { message -> message.id }) { message ->
        MessageRow(message)
    }
}

// Scroll-aware state
val listState = rememberLazyListState()
val showTopButton by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}

// Sticky headers
LazyColumn {
    stickyHeader { DateHeader(date) }
    items(messagesForDate) { MessageRow(it) }
}
```

### 4.3 Compose — Animations
```kotlin
// AnimatedVisibility
AnimatedVisibility(visible = isExpanded) {
    Text("Expanded content")
}

// AnimatedContent with spring transition
AnimatedContent(
    targetState = uiState,
    transitionSpec = {
        fadeIn(spring(stiffness = Spring.StiffnessMediumLow)) togetherWith
        fadeOut(spring(stiffness = Spring.StiffnessMediumLow))
    },
    label = "screen_transition"
) { state -> /* ... */ }

// Shared element transitions
SharedTransitionLayout {
    AnimatedContent(targetState = showDetail) { isDetail ->
        if (isDetail) {
            DetailScreen(Modifier.sharedElement(rememberSharedContentState("hero"), this@AnimatedContent))
        } else {
            ListItem(Modifier.sharedElement(rememberSharedContentState("hero"), this@AnimatedContent))
        }
    }
}
```

### 4.4 Compose — Modifiers
```kotlin
// Correct: hoist Modifier params; never hardcode Modifier in a component's implementation
@Composable
fun MyComponent(modifier: Modifier = Modifier) {
    Box(modifier.padding(16.dp)) { /* ... */ }
}

// Custom modifier with DrawModifier
fun Modifier.customBorder(color: Color) = this.then(
    object : DrawModifier {
        override fun ContentDrawScope.draw() {
            drawContent()
            drawRect(color = color, style = Stroke(width = 2.dp.toPx()))
        }
    }
)

// Avoid: creating Modifiers inside @Composable body during animation
// WRONG — allocates on every frame:
// Box(Modifier.offset(animatedX.value, 0.dp))
// CORRECT — use graphicsLayer or animateOffset:
Box(Modifier.graphicsLayer { translationX = animatedX.value })
```

### 4.5 Compose — Accessibility
```kotlin
// Semantic roles and content descriptions
Image(
    painter = painterResource(R.drawable.icon),
    contentDescription = stringResource(R.string.icon_description)
)

// Merge descendants for a single accessible unit
Row(
    modifier = Modifier.semantics(mergeDescendants = true) {}
) {
    Icon(Icons.Default.Star, contentDescription = null) // null = decorative
    Text("Favorite")
}

// Custom action
Modifier.semantics {
    customActions = listOf(
        CustomAccessibilityAction(label = "Mark as read") { markRead(); true }
    )
}
```

### 4.6 Compose — Tooling / Preview
```kotlin
@Preview(name = "Light", uiMode = Configuration.UI_MODE_NIGHT_NO)
@Preview(name = "Dark", uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun MyComponentPreview() {
    AppTheme { MyComponent() }
}

// PreviewParameter for data-driven previews
class UserPreviewProvider : PreviewParameterProvider<User> {
    override val values = sequenceOf(User("Alice"), User("Bob"))
}

@Preview
@Composable
fun UserCardPreview(@PreviewParameter(UserPreviewProvider::class) user: User) {
    UserCard(user)
}

// LocalInspectionMode — hide live data in preview
val isPreview = LocalInspectionMode.current
if (!isPreview) { /* fetch real data */ }
```

### 4.7 Compose — Interoperability (Views ↔ Compose)
```kotlin
// Compose inside a View-based layout (ComposeView in XML)
class MyFragment : Fragment() {
    override fun onCreateView(...): View = ComposeView(requireContext()).apply {
        setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
        setContent { AppTheme { MyScreen() } }
    }
}

// View inside Compose (AndroidView)
AndroidView(
    factory = { context -> MapView(context).apply { onCreate(null) } },
    update = { mapView -> mapView.getMapAsync { /* configure */ } }
)
```

### 4.8 DataStore
```kotlin
// Preferences DataStore
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
val DARK_THEME_KEY = booleanPreferencesKey("dark_theme")

// Read
val darkTheme: Flow<Boolean> = context.dataStore.data
    .map { prefs -> prefs[DARK_THEME_KEY] ?: false }

// Write
suspend fun setDarkTheme(enabled: Boolean) {
    context.dataStore.edit { prefs -> prefs[DARK_THEME_KEY] = enabled }
}
```

### 4.9 Gradle KTS & Version Catalogs
```toml
# libs.versions.toml
[versions]
composeBom = "2026.05.02"
kotlin = "2.1.0"

[libraries]
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }

[plugins]
android-application = { id = "com.android.application", version = "8.9.0" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```

```kotlin
// build.gradle.kts
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}
dependencies {
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
}
```

---

## 5. How to Navigate the Repo for a Specific Doc Page

1. Identify the doc URL path. Example: `developer.android.com/develop/ui/compose/lists`
2. The domain segment (`compose`) → top-level folder (`compose/`)
3. The topic segment (`lists`) → package (`lists/`)
4. The snippet file: `LazyListSnippets.kt`
5. The `[START android_compose_lists_*]` tags identify individual examples

For non-Compose docs (`developer.android.com/training/data-storage/datastore`) → `datastore/` module.

---

## 6. Reference URLs

| Resource | URL |
|---|---|
| android/snippets repo | https://github.com/android/snippets |
| Compose snippets root | https://github.com/android/snippets/tree/main/compose/snippets/src/main/java/com/example/compose/snippets |
| DataStore snippets | https://github.com/android/snippets/tree/main/datastore |
| Gradle snippets | https://github.com/android/snippets/tree/main/gradle |
| KMP snippets | https://github.com/android/snippets/tree/main/kmp |
| Views snippets | https://github.com/android/snippets/tree/main/views |
| Wear OS snippets | https://github.com/android/snippets/tree/main/wear |
| Android XR snippets | https://github.com/android/snippets/tree/main/xr |
| Compose documentation index | https://developer.android.com/develop/ui/compose/documentation |
