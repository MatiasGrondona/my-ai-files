---
name: m3-expressive
description: >
  Reference skill for generating Android Jetpack Compose UI using Material 3 Expressive
  (androidx.compose.material3 ≥ 1.5.0-alpha21). Use whenever building or modifying Android UI
  with M3 Expressive components, shape morphing, spring-based motion, adaptive layouts, or theme
  setup. Trigger this skill any time the user mentions Material 3 Expressive, M3 Expressive
  components (FloatingToolbar, DockedToolbar, ButtonGroup, SplitButton, LoadingIndicator,
  WavyProgressIndicator, FAB Menu, FlexibleBottomAppBar, expressive ListItem, expressive menus),
  shape morphing, spring animations in Compose, or adaptive navigation layouts — even if they
  don't explicitly say "skill" or "M3 Expressive".
sources:
  - https://m3.material.io/components
  - https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary
  - https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-api-guidelines.md
  - https://github.com/android/nowinandroid
  - https://github.com/android/compose-samples/tree/main/Reply
  - https://github.com/android/snippets
metadata:
  author: Matias
  last-updated: '2026-06-10'
  keywords:
    - material3-expressive
    - jetpack-compose
    - android-ui
    - shape-morphing
    - spring-animation
    - adaptive-layout
    - floating-toolbar
    - docked-toolbar
    - button-group
    - loading-indicator
    - wavy-progress-indicator
    - fab-menu
    - dynamic-color
    - expressive-list-item
    - expressive-menu
    - motion-scheme
---

## Overview

Material 3 Expressive is an **extension** of Material You (Material Design 3), not a new version.
It adds expressive components, shape morphing, and physics-based motion on top of the existing M3
foundation. Requires `androidx.compose.material3 ≥ 1.5.0-alpha21` — **must be pinned explicitly**
as the Compose BOM resolves to stable `1.4.x` which lacks all Expressive APIs.

> **Roadmap:** At Google I/O 2026, Google announced Material Android is going Compose-first.
> M3 Expressive experimental APIs will be promoted to stable when Material Compose 1.5.0 reaches
> stable (later in 2026). Material Views (MDC-Android) 1.14.0 is the final stable Views release.

**Opt-in annotations (as of alpha18):**
- `@ExperimentalMaterial3ExpressiveApi` — used on APIs still in active flux; requires `@OptIn`.
- `@Material3ExpressiveApi` — introduced in alpha18 for more stable Expressive APIs; does **not** require `@OptIn`. Use it on your own composables that wrap Expressive components.

**APIs graduated to stable (no opt-in needed):**
`WavyProgressIndicator`, `expressiveLightColorScheme`, `materialExpressTheme`, `MotionScheme` / `MaterialTheme.motionScheme`, `TopAppBarScrollBehavior` and related scroll behaviors, `Typography` constructors, Slider APIs.

---

## 1. Dependency Setup (`libs.versions.toml`)

```toml
[versions]
composeBom  = "2026.05.02"   # Alpha BOM — use this when pinning alpha material3
material3   = "1.5.0-alpha21"   # MUST pin explicitly — BOM resolves to 1.4.x

[libraries]
compose-bom        = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
# Pin material3 manually — do NOT rely on BOM to resolve to 1.5.x
androidx-material3 = { group = "androidx.compose.material3", name = "material3", version.ref = "material3" }
```

**`build.gradle.kts` (app module):**

```kotlin
dependencies {
    implementation(platform(libs.compose.bom))
    implementation(libs.androidx.material3)  // Expressive — pinned above BOM
}
```

> **Critical:** Never let the BOM manage `material3`. Any `@ExperimentalMaterial3ExpressiveApi`
> usage will fail to compile if resolved to `1.4.x`. The alpha BOM `2026.05.02` picks up
> `1.5.0-alpha21` — use it when targeting the full alpha stack.

---

## 2. Opt-in

As of `1.5.0-alpha18`, M3 Expressive has two annotation tiers:

```kotlin
// @ExperimentalMaterial3ExpressiveApi — still-evolving APIs, requires explicit opt-in
// File-level (preferred for screens with many Expressive components)
@file:OptIn(ExperimentalMaterial3ExpressiveApi::class)

// Or per-composable
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun MyScreen() { ... }

// @Material3ExpressiveApi — more stable Expressive APIs, no opt-in required.
// Apply to your own composables that surface Expressive components:
@Material3ExpressiveApi
@Composable
fun MyExpressiveCard() { ... }
```

When in doubt, check the API reference: if a component carries `@ExperimentalMaterial3ExpressiveApi`,
you need `@OptIn`. If it carries only `@Material3ExpressiveApi`, no opt-in is needed.

---

## 3. Theme Setup (Dynamic Color + Static Fallback)

```kotlin
// ui/theme/Theme.kt
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> darkColorScheme()
        else      -> lightColorScheme()
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,   // define in Type.kt
        shapes = AppShapes,           // define in Shape.kt
        content = content
    )
}
```

**Design system module structure** (NowInAndroid pattern):

```
core/designsystem/
  theme/
    Color.kt       // val md_theme_light_primary = Color(...), dark variants
    Theme.kt       // AppTheme composable
    Type.kt        // AppTypography = Typography(...)
    Shape.kt       // AppShapes = Shapes(...)
  component/
    Button.kt      // Wrapped/custom M3 components
    TopAppBar.kt
```

Always use `MaterialTheme.colorScheme.*`, `MaterialTheme.typography.*`, and `MaterialTheme.shapes.*`
tokens. Never hardcode `Color(0xFF...)` values inside composables.

**Expressive theme variant (alpha15+):** `MaterialTheme` now uses a single `CompositionLocal` for
all theme data. You can access it via `MaterialTheme.LocalMaterialTheme.current` or
`currentValueOf(MaterialTheme.LocalMaterialTheme)` in `CompositionLocalAccessorScope`.

For an M3 Expressive color scheme (graduated to stable in alpha18):

```kotlin
// Static expressive light/dark color schemes (no dynamic color)
MaterialTheme(
    colorScheme = if (darkTheme) expressiveDarkColorScheme() else expressiveLightColorScheme(),
    // or use materialExpressTheme() for the full opinionated expressive theme
    content = content
)
```

---

## 4. Expressive Components

### 4.1 Toolbars — replaces `BottomAppBar` (deprecated)

```kotlin
// DockedToolbar: shorter, more flexible, docked at bottom
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
DockedToolbar(
    modifier = Modifier.fillMaxWidth(),
    floatingActionButton = {
        FloatingActionButton(onClick = { /*...*/ }) {
            Icon(Icons.Default.Add, contentDescription = "Add")
        }
    }
) {
    IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.Search, contentDescription = null) }
    IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.Edit, contentDescription = null) }
    Spacer(Modifier.weight(1f))
    IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.MoreVert, contentDescription = null) }
}

// FloatingToolbar: floating, horizontal or vertical, pairs with FAB
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
var expanded by remember { mutableStateOf(true) }
FloatingToolbar(
    expanded = expanded,
    floatingActionButton = {
        FloatingActionButton(onClick = { expanded = !expanded }) {
            Icon(Icons.Default.Add, contentDescription = null)
        }
    }
) {
    IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.Share, contentDescription = null) }
    IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.Delete, contentDescription = null) }
}

// FlexibleBottomAppBar: scroll-aware, less intrusive
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
val scrollBehavior = FlexibleBottomAppBarDefaults.exitAlwaysScrollBehavior()
Scaffold(
    modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection),
    bottomBar = {
        FlexibleBottomAppBar(scrollBehavior = scrollBehavior) {
            IconButton(onClick = { /*...*/ }) { Icon(Icons.Default.Home, contentDescription = null) }
        }
    }
) { padding -> /* content */ }
```

### 4.2 ButtonGroup and SplitButton

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
// ButtonGroup: related actions as a unit
ButtonGroup {
    Button(onClick = { /*...*/ }) { Text("Day") }
    Button(onClick = { /*...*/ }) { Text("Week") }
    Button(onClick = { /*...*/ }) { Text("Month") }
}

// SplitButton: primary action + secondary overflow
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
SplitButton(
    leadingButton = { Button(onClick = { /*...*/ }) { Text("Save") } },
    trailingButton = {
        IconButton(onClick = { /*...*/ }) {
            Icon(Icons.Default.ArrowDropDown, contentDescription = "More options")
        }
    }
)
```

### 4.3 LoadingIndicator

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
// Use for indeterminate loading states — replaces CircularProgressIndicator in most UX flows
LoadingIndicator()

// Still use LinearProgressIndicator for determinate progress
LinearProgressIndicator(progress = { uploadProgress })
```

### 4.4 FAB Menu

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
var fabExpanded by remember { mutableStateOf(false) }

FloatingActionButtonMenu(
    expanded = fabExpanded,
    button = {
        ToggleFloatingActionButton(
            checked = fabExpanded,
            onCheckedChange = { fabExpanded = it }
        ) {
            Icon(
                imageVector = if (fabExpanded) Icons.Default.Close else Icons.Default.Add,
                contentDescription = if (fabExpanded) "Close menu" else "Open menu"
            )
        }
    }
) {
    FloatingActionButtonMenuItem(
        onClick = { fabExpanded = false },
        icon = { Icon(Icons.Default.Edit, contentDescription = null) },
        text = { Text("Edit") }
    )
    FloatingActionButtonMenuItem(
        onClick = { fabExpanded = false },
        icon = { Icon(Icons.Default.Share, contentDescription = null) },
        text = { Text("Share") }
    )
}
```

### 4.5 WavyProgressIndicator (stable as of alpha18)

Replaces `CircularProgressIndicator` for expressive indeterminate loading. No opt-in required.

```kotlin
// Indeterminate — expressive wavy animation
WavyProgressIndicator()

// Determinate
WavyProgressIndicator(progress = { uploadProgress })
```

### 4.6 Expressive ListItem (alpha11+)

Supports interactions and segmented styling. Additional `ListItemColors` color fields added.

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
ListItem(
    headlineContent = { Text("Title") },
    supportingContent = { Text("Supporting text") },
    leadingContent = { Icon(Icons.Default.Folder, contentDescription = null) },
    trailingContent = { Text("Meta") }
)
```

### 4.7 Expressive Menus (alpha09+)

New toggleable menu items, selectable menu items, menu groups, and popup. All under
`@ExperimentalMaterial3ExpressiveApi`.

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
DropdownMenu(expanded = expanded, onDismissRequest = { expanded = false }) {
    // Selectable menu item
    SelectableMenuItemContainer(
        selected = selectedIndex == 0,
        onClick = { selectedIndex = 0 }
    ) { Text("Option A") }

    // Toggleable menu item
    ToggleableMenuItemContainer(
        checked = isEnabled,
        onCheckedChange = { isEnabled = it }
    ) { Text("Enable feature") }

    MenuGroup { /* group header + items */ }
}
```

**Cascading submenus** (alpha18+): use `DropdownMenuPopupPositionProviders` with
`rememberDropdownMenuPopupPositionProvider` to position submenus relative to their anchor.

---

## 5. Shape Morphing

Shape morphing works on arbitrary polygon forms and is also built into specific components.

**FilterChip / ElevatedFilterChip / InputChip (alpha18):** new overloads with built-in shape
morphing animate on selection changes using M3 Expressive defaults. Prefer these overloads over
manual polygon animation for chips.

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
FilterChip(
    selected = isSelected,
    onClick = { isSelected = !isSelected },
    label = { Text("Filter") },
    // shape morph happens automatically on selection state change
)
```

For custom shape morphing on arbitrary surfaces:

```kotlin
// Expressive shape transitions between polygon forms
val shapeA = remember { RoundedPolygon.rectangle(rounding = CornerRounding(0.3f)) }
val shapeB = remember { RoundedPolygon.circle() }
val morph  = remember { Morph(shapeA, shapeB) }

val progress by animateFloatAsState(
    targetValue = if (isSelected) 1f else 0f,
    animationSpec = spring(stiffness = Spring.StiffnessMedium),
    label = "shape_morph"
)

Box(
    modifier = Modifier
        .size(56.dp)
        .clip(MorphPolygonShape(morph, progress))
        .background(MaterialTheme.colorScheme.primaryContainer)
)
```

---

## 6. Motion — Spring Physics and MotionScheme

**`MaterialTheme.motionScheme`** graduated to stable in alpha15. Use it to access the theme's
standardized motion tokens instead of hardcoding spring values:

```kotlin
// Access motion tokens from the theme (no opt-in needed)
val motionScheme = MaterialTheme.motionScheme
val fastSpatialSpec = motionScheme.fastSpatialSpec<Float>()
val defaultEffectsSpec = motionScheme.defaultEffectsSpec<Float>()
```

For custom animations, always prefer spring-based `AnimationSpec` over `tween`:

```kotlin
// Screen-level transitions
AnimatedContent(
    targetState = uiState,
    transitionSpec = {
        fadeIn(spring(stiffness = Spring.StiffnessMediumLow)) +
        slideInVertically(spring(stiffness = Spring.StiffnessMediumLow)) { it / 4 } togetherWith
        fadeOut(spring(stiffness = Spring.StiffnessMediumLow))
    },
    label = "screen_transition"
) { state -> /* ... */ }

// Component-level (press feedback, expand/collapse)
val scale by animateFloatAsState(
    targetValue = if (isPressed) 0.95f else 1f,
    animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy),
    label = "press_scale"
)

// Shared element transitions (M3 Expressive encourages these)
SharedTransitionLayout {
    AnimatedContent(targetState = selectedItem) { item ->
        // use Modifier.sharedElement() on matching composables
    }
}
```

---

## 7. Standard Scaffold Template

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
@Composable
fun AppScaffold(
    title: String,
    onNavigateUp: (() -> Unit)? = null,
    floatingActionButton: @Composable () -> Unit = {},
    content: @Composable (PaddingValues) -> Unit
) {
    val scrollBehavior = TopAppBarDefaults.pinnedScrollBehavior()

    Scaffold(
        modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection),
        topBar = {
            TopAppBar(
                title = { Text(title) },
                navigationIcon = {
                    if (onNavigateUp != null) {
                        IconButton(onClick = onNavigateUp) {
                            Icon(Icons.AutoMirrored.Default.ArrowBack, contentDescription = "Back")
                        }
                    }
                },
                scrollBehavior = scrollBehavior
            )
        },
        bottomBar = {
            DockedToolbar(modifier = Modifier.fillMaxWidth()) {
                // navigation icons
            }
        },
        floatingActionButton = floatingActionButton
    ) { padding ->
        content(padding)
    }
}
```

---

## 8. Adaptive Navigation Components

Use different navigation components depending on window size class (from Reply sample pattern):

```kotlin
@Composable
fun AppNavLayout(
    windowAdaptiveInfo: WindowAdaptiveInfo = currentWindowAdaptiveInfo(),
    content: @Composable () -> Unit
) {
    val windowSizeClass = windowAdaptiveInfo.windowSizeClass
    when {
        windowSizeClass.isWidthAtLeastBreakpoint(WindowSizeClass.WIDTH_DP_EXPANDED_LOWER_BOUND) -> {
            // Large screens: NavigationDrawer + LargeFloatingActionButton
            PermanentNavigationDrawer(drawerContent = { /* items */ }) { content() }
        }
        windowSizeClass.isWidthAtLeastBreakpoint(WindowSizeClass.WIDTH_DP_MEDIUM_LOWER_BOUND) -> {
            // Medium screens: NavigationRail + FloatingActionButton
            Row {
                NavigationRail { /* items */ }
                content()
            }
        }
        else -> {
            // Compact: NavigationBar + LargeFloatingActionButton (at bottom)
            Column {
                Box(Modifier.weight(1f)) { content() }
                NavigationBar { /* items */ }
            }
        }
    }
}
```

---

## 9. Notable API Changes (alpha19–alpha21)

These are breaking or behaviour-changing updates introduced after alpha18 that affect active development.

**BottomSheet / ModalBottomSheet — `PartiallyExpanded` anchor (alpha21):**
The anchor is no longer removed automatically based on layout conditions. You now control it explicitly:

```kotlin
// alpha21+: use rememberBottomSheetState to configure anchor
val sheetState = rememberBottomSheetState(
    initialValue = SheetValue.PartiallyExpanded,
    // confirmValueChange, skipPartiallyExpanded, etc.
)
ModalBottomSheet(sheetState = sheetState, onDismissRequest = { /* ... */ }) { /* content */ }
```

> Legacy behaviour can be re-enabled via the `isBottomSheetPartiallyExpandedDeterministicEnabled`
> feature flag, or by using the deprecated `rememberModalBottomSheetState` /
> `rememberStandardBottomSheetState` functions.
> `isAnchoredDraggableComponentsAnchorRecoveryEnabled` has been **removed** from feature flags.

**`animateWidth` — `compressionLimit` (alpha21):**
The old `animateWidth` overload without `compressionLimit` is deprecated. Add the parameter:

```kotlin
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
Modifier.animateWidth(
    targetWidth = ...,
    compressionLimit = 8.dp   // max padding compression before item stops squishing
)
```

**`TimePicker` — shape API (alpha21):**
Shapes can now be passed directly to the `TimePicker` component API.

---

## 10. Compose API Naming Conventions

| Pattern | Rule | Example |
|---|---|---|
| Unit composables | `PascalCase` noun | `FancyButton`, `HomeScreen` |
| Composables returning values | `camelCase` verb | `defaultStyle()`, `rememberScrollState()` |
| `remember {}`-based factories | Prefix with `remember` | `rememberScrollState()`, `rememberCoroutineScope()` |
| `CompositionLocal`s | Prefix with `Local` as adjective | `LocalTheme`, `LocalNavEntryOwner` |
| Constants / sealed values / enums | `PascalCase` | `StiffnessMedium`, `DampingRatioHigh` |

Annotate state holder classes and immutable data passed to composables with `@Stable`/`@Immutable`
to enable Compose compiler skipping.

---

## 11. Anti-Patterns

| Anti-pattern | Correct approach |
|---|---|
| `material3:1.4.x` dependency | Pin `material3:1.5.0-alpha21` or newer, separately from BOM |
| `@OptIn(ExperimentalMaterial3ExpressiveApi::class)` missing on experimental APIs | Annotate every file/function using `@ExperimentalMaterial3ExpressiveApi` APIs |
| `BottomAppBar` | `DockedToolbar` or `FlexibleBottomAppBar` |
| `CircularProgressIndicator()` for expressive loading states | `WavyProgressIndicator()` (now stable, no opt-in) |
| `LoadingIndicator()` for all loading states | Use `WavyProgressIndicator()` for expressive flows; `LoadingIndicator()` is still valid |
| Hardcoded colors `Color(0xFF...)` in composables | `MaterialTheme.colorScheme.*` tokens |
| `tween(...)` for interactive transitions | `spring(...)` or `MaterialTheme.motionScheme.*Spec()` |
| Hardcoded spring values everywhere | Prefer `MaterialTheme.motionScheme` tokens for consistency |
| `CAPITALS_AND_UNDERSCORES` constants | `PascalCase` for constants, sealed objects, enum values |
| `@Composable fun renderButton()` | `@Composable fun Button()` — PascalCase noun |
| `@Composable fun ScrollState()` returning a value | `@Composable fun rememberScrollState()` — camelCase with `remember` prefix |
| `TopAppBarDefaults.pinnedScrollBehavior(isAtTop = ...)` | Renamed to `isAtStart` (alpha16+) |
| `animateWidth(targetWidth)` without `compressionLimit` | Use `animateWidth(targetWidth, compressionLimit)` overload (alpha21+, old overload deprecated) |
| `rememberModalBottomSheetState()` to control `PartiallyExpanded` | Use `rememberBottomSheetState()` directly (alpha21+) |

---

## 12. Reference URLs

| Resource | URL |
|---|---|
| M3 Expressive components | https://m3.material.io/components |
| M3 Compose-first announcement (Google I/O 2026) | https://m3.material.io/blog/material-is-compose-first |
| M3 Compose API reference | https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary |
| Compose Material3 release notes | https://developer.android.com/jetpack/androidx/releases/compose-material3 |
| Compose API guidelines | https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-api-guidelines.md |
| NowInAndroid (architecture reference) | https://github.com/android/nowinandroid |
| Compose samples / Reply (adaptive UI) | https://github.com/android/compose-samples/tree/main/Reply |
| Android snippets | https://github.com/android/snippets |
