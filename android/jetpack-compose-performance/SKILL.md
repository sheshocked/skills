---
name: jetpack-compose-performance
description: Diagnose and optimize Jetpack Compose UI recomposition bottlenecks and memory leaks.
category: android
tags: [compose-performance, recomposition, profiling, kotlin, ui]
---

# Jetpack Compose Performance

## When to Use
Use when Compose layouts lag, drop frames, or trigger excess CPU usage during animations or scrolling.

## Prerequisites
- Android Studio Profiler.

## Workflow
1. Use `remember` and `derivedStateOf` to cache expensive computations.
2. Mark data classes as `@Stable` or `@Immutable`.
3. Use lambda-based modifiers (`Modifier.drawBehind`, `Modifier.offset { ... }`) to skip composition phases.

## Key Patterns
```kotlin
@Composable
fun TransactionHistory(transactions: List<Item>, scrollState: ScrollState) {
    // Avoid recomposing on every pixel scroll
    val showFloatingButton by remember {
        derivedStateOf { scrollState.value > 100 }
    }
    
    Box {
        LazyColumn {
            items(transactions, key = { it.id }) { item ->
                TransactionCard(item)
            }
        }
        if (showFloatingButton) {
            FloatingActionButton(onClick = {})
        }
    }
}
```

## Pitfalls
- **Passing raw collections:** Raw `List` causes recomposition because Kotlin collections aren't immutable. Use `@Immutable` wrappers or persistent collections.
- **Excessive derivedStateOf usage:** Using derivedStateOf for simple values that change at the same rate as the source state adds memory overhead without performance benefits.

## Verification
- Turn on Layout Inspector and check the Recomposition Count columns.
- Capture a CPU trace using Profiler to measure compose frame durations.
