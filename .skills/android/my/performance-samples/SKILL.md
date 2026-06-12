---
name: android-performance-samples
description: >
  Reference skill for the official android/performance-samples repository — the canonical source
  for Android performance testing patterns using Macrobenchmark, Microbenchmark, Baseline Profiles,
  and JankStats. Use when writing or reviewing benchmark tests, generating Baseline Profiles,
  measuring app startup / scroll jank / frame timing, benchmarking isolated Kotlin/Java code, or
  integrating benchmarks into CI with Firebase Test Lab. Trigger whenever the user mentions
  benchmarking, performance testing, startup time, frame jank, Baseline Profiles, JankStats,
  MacrobenchmarkRule, BenchmarkRule, CompilationMode, or composition tracing — even if they don't
  explicitly name the performance-samples repo.
sources:
  - https://github.com/android/performance-samples
metadata:
  author: Matias
  last-updated: '2026-06-10'
  keywords:
    - macrobenchmark
    - microbenchmark
    - baseline-profiles
    - jankstats
    - startup-timing
    - frame-timing
    - composition-tracing
    - performance-testing
    - firebase-test-lab
    - compilation-mode
    - benchmark-rule
    - trace-section
---

## Overview

`android/performance-samples` contains three standalone Android projects, each targeting a distinct
layer of performance measurement. They are the official references for how Google intends these
libraries to be used in production apps.

| Sample | Library | What it measures |
|---|---|---|
| `MacrobenchmarkSample` | `androidx.benchmark:benchmark-macro` | Full app user journeys: startup, scroll, jank, Baseline Profile generation |
| `MicrobenchmarkSample` | `androidx.benchmark:benchmark-junit4` | Isolated Kotlin/Java code: algorithms, serialization, UI inflation |
| `JankStatsSample` | `androidx.metrics:metrics-performance` | Runtime jank detection and reporting in production builds |

> **Device requirement:** Always run benchmarks on a physical device with the screen on.
> Emulator results are unreliable due to inconsistent CPU scheduling. Unlock the device and
> disable always-on display before running.

---

## 1. MacrobenchmarkSample

**What it covers:** Application-level performance — the things real users perceive. Runs in a
separate process from the app under test to simulate cold/warm/hot starts and scrolling flows.

### 1.1 Project Structure

```
MacrobenchmarkSample/
  app/                          // Target app under test (buildType: benchmark)
    src/main/baselineProfiles/  // Generated Baseline Profile files (AGP 8.0+)
    AndroidManifest.xml         // Must declare <profileable android:shell="true"/>
  macrobenchmark/               // Benchmark module (separate process)
    src/androidTest/
      StartupBenchmarks.kt      // Cold/warm/hot startup + CompilationMode comparison
      ScrollBenchmarks.kt       // LazyColumn / RecyclerView scroll frame timing
      BaselineProfileGenerator.kt // Generates the Baseline Profile
      baselineBenchmarks/       // baseBenchmarks drop-in library (copy to your app)
  ftl/                          // Firebase Test Lab CI workflow
```

### 1.2 Startup Benchmark

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class StartupBenchmarks {

    @get:Rule
    val rule = MacrobenchmarkRule()

    // Measure cold start with no pre-compilation (worst case)
    @Test
    fun startupCompilationNone() = startup(CompilationMode.None())

    // Measure cold start with Baseline Profiles applied (best case)
    @Test
    fun startupCompilationBaselineProfiles() = startup(CompilationMode.Partial())

    private fun startup(compilationMode: CompilationMode) {
        rule.measureRepeated(
            packageName = "com.example.macrobenchmark",
            metrics = listOf(StartupTimingMetric()),
            compilationMode = compilationMode,
            startupMode = StartupMode.COLD,
            iterations = 10,
            setupBlock = { pressHome() }
        ) {
            startActivityAndWait()
            // Wait for content to be visible before ending measurement
            device.wait(Until.hasObject(By.res("content")), 5_000)
        }
    }
}
```

### 1.3 Scroll / Frame Timing Benchmark

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class ScrollBenchmarks {

    @get:Rule
    val rule = MacrobenchmarkRule()

    @Test
    fun scrollCompilationNone() = scroll(CompilationMode.None())

    @Test
    fun scrollCompilationBaselineProfiles() = scroll(CompilationMode.Partial())

    private fun scroll(compilationMode: CompilationMode) {
        rule.measureRepeated(
            packageName = "com.example.macrobenchmark",
            metrics = listOf(FrameTimingMetric()),
            compilationMode = compilationMode,
            startupMode = StartupMode.WARM,
            iterations = 5,
            setupBlock = {
                startActivityAndWait()
            }
        ) {
            // Scroll the list using UiAutomator
            val list = device.findObject(By.res("lazy_list"))
            list.setGestureMargin(device.displayWidth / 5)
            list.fling(Direction.DOWN)
            device.waitForIdle()
        }
    }
}
```

### 1.4 Baseline Profile Generation

Baseline Profiles pre-compile critical code paths at install time, reducing JIT compilation during
startup and first-run scrolls. AGP 8.0+ supports multiple profile files stored in
`src/main/baselineProfiles/`.

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class BaselineProfileGenerator {

    @get:Rule
    val rule = BaselineProfileRule()

    // Startup journey — generates startup profile
    @Test
    fun generateStartup() {
        rule.collect(packageName = "com.example.macrobenchmark") {
            pressHome()
            startActivityAndWait()
        }
    }

    // Scroll journey — generates profile covering list rendering code paths
    @Test
    fun generateScrolling() {
        rule.collect(packageName = "com.example.macrobenchmark") {
            pressHome()
            startActivityAndWait()
            val list = device.findObject(By.res("lazy_list"))
            list.fling(Direction.DOWN)
            device.waitForIdle()
        }
    }
}
```

**`AndroidManifest.xml` target app requirement:**
```xml
<application>
    <profileable android:shell="true" tools:targetApi="r"/>
</application>
```

**`build.gradle.kts` target app:**
```kotlin
android {
    buildTypes {
        create("benchmark") {
            initWith(buildTypes.getByName("release"))
            signingConfig = signingConfigs.getByName("debug")
            matchingFallbacks += listOf("release")
            isDebuggable = false
        }
    }
}
```

### 1.5 Custom Trace Sections + Composition Tracing

Add custom `Trace` sections to the app under test, then capture them in the benchmark:

```kotlin
// In the app — wrap any expensive operation
Trace.beginSection("LoadingContent")
try {
    loadHeavyContent()
} finally {
    Trace.endSection()
}

// In the benchmark — capture with TraceSectionMetric
rule.measureRepeated(
    packageName = TARGET_PACKAGE,
    metrics = listOf(
        FrameTimingMetric(),
        TraceSectionMetric("LoadingContent"),
    ),
    iterations = 5,
) { /* interaction */ }
```

**Composition Tracing** — records which Composables recompose and for how long. Requires
`androidx.benchmark.perfettoSdkTracing.enable=true` instrumentation argument. Use the
`Scroll List With Composition Tracing` run configuration in the sample project.

### 1.6 CompilationMode Reference

| Mode | Meaning | When to use |
|---|---|---|
| `CompilationMode.None()` | No pre-compilation; all JIT at runtime | Baseline measurement (worst case) |
| `CompilationMode.Partial()` | Uses installed Baseline Profile if present | Measure profile benefit |
| `CompilationMode.Full()` | Full AOT compilation (like `adb shell cmd package compile`) | Theoretical best case ceiling |
| `CompilationMode.Ignore()` | Don't change compilation state | CI where you pre-install a specific compilation |

### 1.7 Firebase Test Lab CI

The sample includes a complete GitHub Actions workflow in `ftl/` for running Macrobenchmarks
on Firebase Test Lab on every push to the `macrobenchmark` branch. Key steps:
1. Build `app-benchmark.apk` and `macrobenchmark.apk`
2. Upload both to FTL via `gcloud firebase test android run --type instrumentation`
3. Pull results and artifacts from GCS bucket

---

## 2. MicrobenchmarkSample

**What it covers:** Sub-millisecond timing of isolated Kotlin/Java code — algorithms, data
structure operations, serialization, UI inflation. Runs in the same process. Results are in
nanoseconds. Requires a separate `microbenchmark` module with `debuggable = false`.

### 2.1 Project Structure

```
MicrobenchmarkSample/
  app/                          // Library module with the code being benchmarked
    src/main/java/
      SortingAlgorithms.kt      // Example: algorithms under test
      BitmapUtils.kt            // Example: bitmap operations under test
  microbenchmark/               // Benchmark module
    build.gradle.kts            // testInstrumentationRunner = AndroidBenchmarkRunner
                                // android.defaultConfig.testInstrumentationRunnerArguments
                                //   ["androidx.benchmark.suppressErrors"] = "EMULATOR"
    src/androidTest/
      SortingAlgorithmsBenchmark.kt
      BitmapBenchmark.kt
```

### 2.2 Basic Microbenchmark Pattern

```kotlin
@RunWith(AndroidJUnit4::class)
class SortingAlgorithmsBenchmark {

    // Use the same random seed for reproducibility
    private val unsorted = IntArray(10_000).also { array ->
        val random = Random(0)
        array.indices.forEach { i -> array[i] = random.nextInt() }
    }

    @get:Rule
    val benchmarkRule = BenchmarkRule()

    @Test
    fun benchmarkQuickSort() {
        var listToSort: IntArray
        benchmarkRule.measureRepeated {
            // runWithTimingDisabled: setup work outside the measured window
            listToSort = runWithTimingDisabled { unsorted.copyOf() }
            SortingAlgorithms.quickSort(listToSort)
        }
        // assert outside the benchmark loop to avoid measurement overhead
        assertTrue(listToSort.isSorted)
    }
}
```

**Key rules:**
- Put setup code inside `runWithTimingDisabled {}` — allocations and copies that aren't part of
  what you're measuring.
- Keep only the operation under test inside the measured block.
- Assert once after `measureRepeated`, never inside it.
- Set `debuggable = false` in the benchmark module — the library enforces this; results on a
  debuggable build are rejected with a warning.

### 2.3 `build.gradle.kts` for Microbenchmark Module

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
}

android {
    defaultConfig {
        testInstrumentationRunner = "androidx.benchmark.junit4.AndroidBenchmarkRunner"
        // Suppress emulator errors if running on CI — remove for local device runs
        testInstrumentationRunnerArguments["androidx.benchmark.suppressErrors"] = "EMULATOR"
    }
    buildTypes {
        debug {
            // Required: benchmarks must not be debuggable
            isDebuggable = false
        }
    }
}

dependencies {
    androidTestImplementation(libs.androidx.benchmark.junit4)
    androidTestImplementation(libs.junit)
    androidTestImplementation(libs.androidx.test.runner)
}
```

### 2.4 Running Microbenchmarks

```bash
# From terminal — runs all benchmarks in the microbenchmark module
./gradlew microbenchmark:cC

# Or run a single benchmark class from Android Studio:
# Right-click the test class → Run (with screen on, real device)
```

Results appear in the **Run** tab in Android Studio and are also written to a JSON file in
`/data/data/<benchmark_package>/files/` on the device.

---

## 3. JankStatsSample

**What it covers:** Runtime jank detection in **production builds** — not a testing tool, but a
monitoring library that reports slow frames to your analytics backend as users interact with the
app. Designed to complement Macrobenchmark (which catches jank in CI) with live field data.

### 3.1 Project Structure

```
JankStatsSample/
  app/src/main/java/com/example/jankstats/
    JankLoggingActivity.kt      // Activity that creates and starts JankStats
    JankStatsAggregator.kt      // Aggregates frame data and reports on lifecycle events
    PerStateIssueReporter.kt    // Custom OnFrameListener implementation
```

### 3.2 Setup and Usage

```kotlin
class JankLoggingActivity : AppCompatActivity() {

    private lateinit var jankStats: JankStats

    // Custom listener: receives per-frame performance data
    private val jankFrameListener = JankStats.OnFrameListener { frameData ->
        if (frameData.isJank) {
            // frameData.frameDurationUiNanos — actual UI thread duration
            // frameData.states — list of PerformanceMetricsState entries at the time of the frame
            Log.w("JankStats", "Janky frame: ${frameData.frameDurationUiNanos / 1_000_000}ms " +
                "states=${frameData.states}")
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Create JankStats for this window; pass the listener
        jankStats = JankStats.createAndTrack(window, jankFrameListener)
    }

    override fun onResume() {
        super.onResume()
        jankStats.isTrackingEnabled = true
    }

    override fun onPause() {
        super.onPause()
        jankStats.isTrackingEnabled = false
    }
}
```

### 3.3 Annotating UI State for Jank Attribution

Tag frames with what the user was doing so you can correlate jank to specific interactions:

```kotlin
// Get the PerformanceMetricsState holder for this window
val metricsStateHolder = PerformanceMetricsState.getHolderForHierarchy(binding.root)

// Set state before the interaction starts
metricsStateHolder.state?.putState("RecyclerView", "Scrolling")

// Clear state when the interaction ends
metricsStateHolder.state?.removeState("RecyclerView")
```

With this, each `FrameData` delivered to the `OnFrameListener` includes the states active at
frame time, so you can attribute "janky frame occurred during RecyclerView scrolling" in your
analytics.

---

## 4. Decision Guide: Which Tool to Use

| Question | Answer |
|---|---|
| "Is my app's startup fast enough?" | Macrobenchmark — `StartupTimingMetric` + `CompilationMode` comparison |
| "Is my scroll smooth (< 16ms per frame)?" | Macrobenchmark — `FrameTimingMetric` on a scrolling interaction |
| "What's the benefit of my Baseline Profile?" | Macrobenchmark — compare `CompilationMode.None()` vs `CompilationMode.Partial()` |
| "Is my sorting/parsing/caching code fast?" | Microbenchmark — `BenchmarkRule.measureRepeated` |
| "Which of two algorithms is faster?" | Microbenchmark — two `@Test` methods, compare ns output |
| "Are real users seeing jank in production?" | JankStats — `OnFrameListener` reporting to your analytics |
| "What Composables are causing recomposition jank?" | Macrobenchmark + Composition Tracing + Perfetto |

---

## 5. Common Anti-Patterns

| Anti-pattern | Correct approach |
|---|---|
| Running benchmarks on an emulator | Use a physical device with screen on |
| `debuggable = true` in benchmark module | Set `isDebuggable = false` — library rejects results otherwise |
| Setup/teardown inside `measureRepeated {}` | Wrap setup in `runWithTimingDisabled {}` |
| Asserting inside the benchmark loop | Assert once after `measureRepeated` |
| Only measuring `CompilationMode.Full()` | Always include `CompilationMode.None()` as baseline |
| Not declaring `<profileable>` in app manifest | Required for Baseline Profile generation and Macrobenchmark |
| Storing Baseline Profiles outside `src/main/baselineProfiles/` | AGP 8.0+ expects this path; supports multiple files |
| Using Macrobenchmark for algorithm-level timing | Use Microbenchmark — Macrobenchmark overhead is too coarse (ms, not ns) |

---

## 6. Reference URLs

| Resource | URL |
|---|---|
| performance-samples repo | https://github.com/android/performance-samples |
| MacrobenchmarkSample | https://github.com/android/performance-samples/tree/main/MacrobenchmarkSample |
| MicrobenchmarkSample | https://github.com/android/performance-samples/tree/main/MicrobenchmarkSample |
| JankStatsSample | https://github.com/android/performance-samples/tree/main/JankStatsSample |
| Macrobenchmark guide | https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview |
| Macrobenchmark metrics reference | https://developer.android.com/topic/performance/benchmarking/macrobenchmark-metrics |
| Microbenchmark guide | https://developer.android.com/topic/performance/benchmarking/microbenchmark-overview |
| Microbenchmark write guide | https://developer.android.com/topic/performance/benchmarking/microbenchmark-write |
| Baseline Profiles guide | https://developer.android.com/topic/performance/baselineprofiles/overview |
| JankStats guide | https://developer.android.com/topic/performance/jankstats |
| Android performance overview | https://developer.android.com/topic/performance |
