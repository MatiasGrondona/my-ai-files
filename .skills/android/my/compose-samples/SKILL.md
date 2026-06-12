---
name: compose-samples
description: >
  Reference skill for finding real-world Jetpack Compose implementation patterns across the
  official Android sample repos. Use when looking for canonical examples of architecture,
  UI patterns, theming, animations, adaptive layouts, testing, or advanced graphics in Compose.
  Trigger whenever the user asks how something is done "in practice", references a specific
  sample app (Reply, Jetcaster, JetLagged, JetNews, Jetchat, Jetsnack), asks about
  NowInAndroid architecture or modularization, or wants a production-quality reference for
  any Compose or Material 3 pattern — even if they don't explicitly name a sample repo.
sources:
  - https://github.com/android/compose-samples
  - https://github.com/android/nowinandroid
  - https://m3.material.io/components
metadata:
  author: Matias
  last-updated: '2026-06-10'
  keywords:
    - jetpack-compose
    - android-samples
    - material3
    - adaptive-layout
    - architecture
    - modularization
    - nowinandroid
    - reply
    - jetcaster
    - jetlagged
    - jetnews
    - jetchat
    - jetsnack
    - animations
    - custom-layouts
    - ui-testing
---

## Overview

Two authoritative reference repositories maintained by Google for production-quality Jetpack
Compose patterns:

- **`android/compose-samples`** — six standalone sample apps, each targeting a distinct complexity
  level and feature set. Clone the whole repo and open individual sub-projects in Android Studio.
- **`android/nowinandroid`** — a fully functional, production-shipped app covering the official
  Android architecture guidance end-to-end, with deep modularization, testing, and real data.

Use these repos as the ground truth when generating Compose code. Always prefer patterns found
here over generic implementations.

---

## 1. Sample App Catalogue (`android/compose-samples`)

### 1.1 Reply — Adaptive Layout & Material 3 Reference
**Complexity:** Medium | **Primary use:** Adaptive UI, M3 theming, navigation components

The canonical reference for adaptive layouts across phones, tablets, and foldables. Implements
Material Design 3 theming, dynamic color, and all three adaptive navigation components
(`NavigationBar`, `NavigationRail`, `NavigationDrawer`) with window size class switching.

```
compose-samples/Reply/
  app/src/main/java/com/example/reply/
    ui/
      ReplyApp.kt              // top-level adaptive layout switching
      ReplyHomeScreen.kt       // adaptive content pane
      navigation/
        ReplyNavigationComponents.kt  // NavigationBar / Rail / Drawer
    data/                      // fake repository, email data model
```

**Key patterns to copy:**
- `WindowAdaptiveInfo` + `currentWindowAdaptiveInfo()` for breakpoint switching
- `NavigationSuiteScaffold` from `material3-adaptive-navigation-suite`
- M3 dynamic color: `dynamicDarkColorScheme` / `dynamicLightColorScheme`
- `SharedTransitionLayout` + `Modifier.sharedElement()` for list-to-detail

**Direct link:** https://github.com/android/compose-samples/tree/main/Reply

---

### 1.2 Jetcaster — Advanced Architecture & Dynamic Theming
**Complexity:** Advanced | **Primary use:** Redux-style arch, dynamic color from images, Room, Coroutines

Full-featured podcast app. The architecture reference for unidirectional data flow with a
Redux-style store. Demonstrates extracting dominant colors from artwork to drive the UI theme at
runtime — the key pattern for dynamic theming beyond M3 dynamic color.

```
compose-samples/Jetcaster/
  app/src/main/java/com/example/jetcaster/
    Graph.kt                   // manual DI wiring
    ui/
      home/HomeViewModel.kt    // UDF: StateFlow<HomeViewState>
    data/
      PodcastStore.kt          // Repository with Room + Coroutines
      PodcastFetcher.kt        // network + RSS parsing
```

**Key patterns to copy:**
- `StateFlow<UiState>` ViewModel pattern with sealed `Result` wrappers
- `WindowInsets` edge-to-edge handling
- Dominant color extraction from `Bitmap` → `MaterialTheme` override
- Room + Coroutines for local persistence

**Direct link:** https://github.com/android/compose-samples/tree/main/Jetcaster

---

### 1.3 JetLagged — Custom Graphics & Animations
**Complexity:** Medium | **Primary use:** Custom layouts, Canvas drawing, AGSL shaders, animations

Sleep tracking app. The reference for anything beyond standard components: custom `Canvas`-based
charts, `Path` drawing, `AGSL` (Android Graphics Shading Language) shaders, and complex
transition animations. Also demonstrates `Modifier.animateBounds()` and gesture-driven drawers.

```
compose-samples/JetLagged/
  app/src/main/java/com/example/jetlagged/
    SleepBar.kt                // custom animated bar chart with transitions
    JetLaggedDrawer.kt         // gesture-driven drawer, PredictiveBackHandler
    ui/theme/                  // custom color/typography tokens
```

**Key patterns to copy:**
- `DrawScope` + `Path` for custom chart rendering
- `AGSL` shader via `RuntimeShader` + `ShaderBrush`
- `updateTransition` for coordinated multi-property animations
- `PredictiveBackHandler` for gesture-based navigation
- `Modifier.animateBounds()` for layout-change animations

**Direct link:** https://github.com/android/compose-samples/tree/main/JetLagged

---

### 1.4 JetNews — Typical Material App & UI Testing
**Complexity:** Medium | **Primary use:** Standard M3 patterns, resource loading, UI tests

Blog reader app. The starting reference for a typical Material app with real-world architecture
and light/dark theming. The best source for UI testing patterns with `ComposeTestRule`.

```
compose-samples/JetNews/
  app/src/main/java/com/example/jetnews/
    ui/
      home/HomeScreen.kt       // list + detail with back stack
      article/ArticleScreen.kt
    data/
      posts/PostsRepository.kt // fake repository
  app/src/androidTest/         // UI tests with ComposeTestRule
```

**Key patterns to copy:**
- `Scaffold` + `TopAppBar` + `SnackbarHost` standard layout
- `rememberLazyListState()` scroll-aware top bar
- `ComposeTestRule` — `onNodeWithText`, `performClick`, `assertIsDisplayed`
- Light/dark theme switching via `isSystemInDarkTheme()`

**Direct link:** https://github.com/android/compose-samples/tree/main/JetNews

---

### 1.5 Jetchat — UI State & Text Input
**Complexity:** Low | **Primary use:** Chat UI, text input, ViewModel state, Fragment interop

Chat app focused on `TextField` state management, keyboard handling, and back button behavior.
Also demonstrates mixing Compose with Fragments and integrating Architecture Components
(`ViewModel`, `LiveData`) in a hybrid setup.

```
compose-samples/Jetchat/
  app/src/main/java/com/example/jetchat/
    conversation/
      ConversationFragment.kt  // Fragment hosting Compose
      ConversationContent.kt   // message list + input row
      UserInput.kt             // multi-state text input with IME handling
    profile/ProfileFragment.kt
```

**Key patterns to copy:**
- `ImeAction` + `KeyboardOptions` for text input flows
- `BackHandler` for custom back navigation
- `LiveData.observeAsState()` bridge from ViewModel to Compose
- M3 dynamic color with Material You

**Direct link:** https://github.com/android/compose-samples/tree/main/Jetchat

---

### 1.6 Jetsnack — Custom Design System
**Complexity:** Medium | **Primary use:** Custom theme/design system, custom layouts, animations

Snack ordering app with a fully custom design system that deliberately bypasses standard M3
components. The reference when building branded apps with custom tokens, shapes, and gradients
that diverge from Material defaults.

```
compose-samples/Jetsnack/
  app/src/main/java/com/example/jetsnack/
    ui/
      theme/
        Theme.kt               // custom JetsnackTheme (not MaterialTheme)
        Color.kt               // custom color tokens
      components/
        Gradient.kt            // custom gradient modifiers
        Filters.kt             // horizontal scrolling filter chips
      snackdetail/SnackDetail.kt  // collapsing header with custom animations
```

**Key patterns to copy:**
- `CompositionLocalProvider` for a custom design system parallel to `MaterialTheme`
- `Modifier.offsetGradientBackground()` — custom gradient Modifier
- Collapsing `TopAppBar` via `nestedScroll` + manual `graphicsLayer` alpha
- Custom layout with `Layout {}` composable

**Direct link:** https://github.com/android/compose-samples/tree/main/Jetsnack

---

## 2. NowInAndroid (`android/nowinandroid`)

The production reference for **everything at scale**: multi-module architecture, offline-first
data, Hilt DI, full test coverage, convention plugins, and adaptive UI. The `compose-samples`
apps are optimized for teaching individual patterns; NowInAndroid is optimized for showing how
all patterns fit together in a real shipped app.

### 2.1 Module Structure

```
nowinandroid/
  app/                         // app module — DI wiring, MainActivity
  core/
    designsystem/              // AppTheme, tokens, shared components
    data/                      // repositories (interface + impl)
    database/                  // Room DAOs, entities
    datastore/                 // Proto DataStore user prefs
    network/                   // Retrofit, serialization
    model/                     // shared data classes
    ui/                        // shared UI components (NewsFeed, etc.)
    common/                    // pure Kotlin utils
    testing/                   // shared test fakes and utilities
  feature/
    foryou/                    // For You screen (api + impl)
    bookmarks/                 // Bookmarks screen (api + impl)
    search/                    // Search screen (api + impl)
    interests/                 // Interests screen (api + impl)
    topic/                     // Topic detail (api + impl)
    settings/                  // Settings bottom sheet (impl only)
  sync/work/                   // WorkManager background sync
  build-logic/                 // convention plugins (shared Gradle config)
```

### 2.2 Architecture Patterns

**ViewModel → UiState flow:**
```kotlin
// Each feature follows this pattern exactly
data class ForYouUiState(
    val feedState: NewsFeedUiState,
    val onboardingUiState: OnboardingUiState,
)

class ForYouViewModel @Inject constructor(
    private val userDataRepository: UserDataRepository,
    newsRepository: NewsRepository,
) : ViewModel() {

    val uiState: StateFlow<ForYouUiState> = combine(
        userDataRepository.userData,
        newsRepository.getNewsResources(),
        ::ForYouUiState
    ).stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5_000),
        initialValue = ForYouUiState(...)
    )
}
```

**Hilt injection pattern:**
```kotlin
// Interface in :core:data
interface NewsRepository {
    fun getNewsResources(): Flow<List<NewsResource>>
}

// Impl in :core:data (bound via @Binds in a Hilt module)
class OfflineFirstNewsRepository @Inject constructor(
    private val newsResourceDao: NewsResourceDao,
    private val network: NiaNetworkDataSource,
) : NewsRepository { ... }
```

**Convention plugin pattern** (`build-logic/`):
```kotlin
// Instead of repeating compose setup in every module:
// build.gradle.kts
plugins {
    alias(libs.plugins.nowinandroid.android.library.compose)
}
// This plugin applies composeOptions, dependencies, compiler args automatically
```

### 2.3 Testing Patterns

NowInAndroid has three layers of tests — all fakes live in `:core:testing`:

```kotlin
// Fake repository for unit tests (no Room/network needed)
class TestNewsRepository : NewsRepository {
    private val newsFlow = MutableSharedFlow<List<NewsResource>>()
    suspend fun sendNewsResources(list: List<NewsResource>) = newsFlow.emit(list)
    override fun getNewsResources() = newsFlow.asSharedFlow()
}

// ViewModel unit test
@Test
fun uiState_whenFollowedTopicsAndNewsLoaded_thenShowFeed() = runTest {
    val viewModel = ForYouViewModel(TestUserDataRepository(), TestNewsRepository())
    testNewsRepository.sendNewsResources(testNewsResources)
    assertEquals(NewsFeedUiState.Success(testNewsResources), viewModel.uiState.value.feedState)
}
```

**Screenshot tests** via Roborazzi are included in NowInAndroid as a reference for screenshot
regression testing with Compose.

---

## 3. Pattern-to-Sample Quick Reference

| Need | Best sample | Key file |
|---|---|---|
| Adaptive layout (phone/tablet/foldable) | Reply | `ReplyApp.kt` |
| M3 dynamic color | Reply / Jetchat | `Theme.kt` |
| List-to-detail shared element transition | Reply | `ReplyHomeScreen.kt` |
| Redux-style UDF architecture | Jetcaster | `HomeViewModel.kt` |
| Dynamic color from image/artwork | Jetcaster | `DynamicThemePrimaryColorsContainer.kt` |
| Custom Canvas chart / Path drawing | JetLagged | `SleepBar.kt` |
| AGSL shader | JetLagged | `ui/theme/` |
| `PredictiveBackHandler` gesture drawer | JetLagged | `JetLaggedDrawer.kt` |
| Standard Scaffold + TopAppBar pattern | JetNews | `HomeScreen.kt` |
| Compose UI testing (`ComposeTestRule`) | JetNews | `androidTest/` |
| TextField / IME / keyboard handling | Jetchat | `UserInput.kt` |
| Fragment + Compose interop | Jetchat | `ConversationFragment.kt` |
| Custom design system (non-M3 tokens) | Jetsnack | `Theme.kt` |
| Custom `Layout {}` composable | Jetsnack | `Filters.kt` |
| Multi-module architecture at scale | NowInAndroid | `core/` + `feature/` |
| Hilt DI + repository pattern | NowInAndroid | `core/data/` |
| Convention Gradle plugins | NowInAndroid | `build-logic/` |
| ViewModel `StateFlow` + `stateIn` pattern | NowInAndroid | `ForYouViewModel.kt` |
| Fake repositories for unit testing | NowInAndroid | `core/testing/` |
| Screenshot regression tests (Roborazzi) | NowInAndroid | `screenshotTest/` |

---

## 4. Reference URLs

| Resource | URL |
|---|---|
| compose-samples repo | https://github.com/android/compose-samples |
| Reply sample | https://github.com/android/compose-samples/tree/main/Reply |
| Jetcaster sample | https://github.com/android/compose-samples/tree/main/Jetcaster |
| JetLagged sample | https://github.com/android/compose-samples/tree/main/JetLagged |
| JetNews sample | https://github.com/android/compose-samples/tree/main/JetNews |
| Jetchat sample | https://github.com/android/compose-samples/tree/main/Jetchat |
| Jetsnack sample | https://github.com/android/compose-samples/tree/main/Jetsnack |
| NowInAndroid repo | https://github.com/android/nowinandroid |
| NowInAndroid architecture journey | https://github.com/android/nowinandroid/blob/main/docs/ArchitectureLearningJourney.md |
| NowInAndroid modularization journey | https://github.com/android/nowinandroid/blob/main/docs/ModularizationLearningJourney.md |
| M3 components reference | https://m3.material.io/components |
