---
name: compose-performance
description: 
category: android
tags: [compose-performance]
---

## When to Use
Use this skill when diagnosing recomposition storms, optimizing Compose performance, analyzing unstable types, or setting up baseline profiles.

## Core Concepts
- **Recomposition count**: Each composable called every time state it reads changes
- **Stable types**: Types the compiler can skip recomposition for
- **Deferred reads**: Using lambda captures vs property access
- **Baseline profile**: AOT compilation hint for first-use performance

## Diagnostic Workflow
1. **Enable recomposition counts**: Settings > Developer Options > Show Recomposition Counts
2. **Compose Compiler Metrics**: ./gradlew assembleRelease -PenableComposeCompilerReports=true
3. **Analyze reports**: Find unstable types and frequent recompositions
4. **Fix**: Add @Immutable/@Stable, use derivedStateOf, move state down

## Key Fixes
```kotlin
// BEFORE: Unstable — recomposes every frame
@Composable
fun BadList(items: List<Pair<String, Int>>)  // List is unstable!

// AFTER: Stable
@Composable
fun GoodList(items: List<ListItem>)  // ListItem is data class with val fields

// BEFORE: Recomposition storm
@Composable
fun Expensive(@Composable content: () -> Unit) {
    content()  // Inline recomposition
}

// AFTER: Move content to stable slot
@Composable
fun Good(@Composable content: () -> Unit) {
    Box { content() }  // content is captured as Stable lambda
}

// BEFORE: Reading state in layout
@Composable
fun Bad() {
    val size = remember { mutableIntStateOf(0) }
    LaunchedEffect(Unit) {
        size.intValue = 100
    }
    Box(modifier = Modifier.size(size.intValue.dp))  // Recomposes on every size change
}

// AFTER: Use derivedStateOf
@Composable
fun Good() {
    val size = remember { mutableIntStateOf(0) }
    val stableSize by remember { derivedStateOf { size.intValue } }
    Box(modifier = Modifier.size(stableSize.dp))
}
```

## Baseline Profiles
```bash
# Generate baseline profile
./gradlew :app:assembleRelease -PenableBaselineProfile

# Install on device
adb install -r app/build/outputs/apk/release/app-release.apk
```

## Pitfalls
- **lambda{} in parameters**: Lambda parameters create unstable closures
- **Collections**: List, Set, Map are unstable — use ImmutableList from kotlinx.collections
- **enum/sealed**: Enums are stable; sealed classes with stable subtypes are stable
- **@Stable annotation**: Only use when type is actually stable
- **BoxWithConstraints**: Triggers extra layout measurement

## Verification
- Recomposition counts should be 0-1 for screens after initial composition
- Compose Compiler metrics show 0 unstable parameters in critical paths
- Baseline profile reduces first-frame rendering time by 30-60%