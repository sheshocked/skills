---
name: jetpack-compose-ui
description: 
category: android
tags: [jetpack-compose-ui]
---

## When to Use
Use this skill when building production Android UIs with Jetpack Compose, managing state, handling recomposition, creating custom layouts, or implementing animations.

## Core Concepts
- **Recomposition**: Recalling composables when state changes
- **State hoisting**: Move state up, events down (unidirectional flow)
- **Remember/Savable**: Preserve state across recompositions
- **Side effects**: LaunchedEffect, DisposableEffect, SideEffect, derivedStateOf
- **Slot APIs**: Content lambdas for flexible compositions

## Workflow
1. **State placement**: State lives in the lowest common ancestor of affected composables
2. **Derived state**: Use derivedStateOf() when computed state depends on other state
3. **Recomposition**: Minimize by ensuring stable parameters and stable types
4. **Side effects**: Use LaunchedEffect for one-time work, rememberCoroutineScope for event handlers

## Key Patterns
```kotlin
// State hoisting
@Composable
fun Counter(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) { Text("$count") }
}

// Derived state
val filteredItems = remember {
    derivedStateOf { items.filter { it.name.contains(query) } }
}

// LaunchedEffect for one-time work
LaunchedEffect(Unit) {
    analytics.trackScreen("HomeScreen")
}

// Recomposition stability
data class User(val name: String)  // stable if all props are val
@Composable
fun UserCard(user: User)  // Won't recompose if user is stable
```

## Performance Rules
1. **Stable types**: Use data classes with val properties
2. **Unstable parameters**: Use @Immutable or @Stable annotations
3. **LazyColumn**: Use key parameter for efficient list diffing
4. **rememberCache**: Cache expensive computations with remember
5. **Layout pass**: Minimize nested layouts; use BoxWithConstraints sparingly

## Pitfalls
- **MutableState hoisting**: Don't hoist State<T> — hoist the value, not the state wrapper
- **Side effects in composition**: Never start async work directly in Composable; use LaunchedEffect
- **remember key**: Always provide keys when condition changes: remember(key) { ... }
- **Preview**: @Preview composables don't have lifecycle; test with @Composable testing
- **Diff tool**: Use Compose compiler metrics to detect unstable types

## Verification
- Use Compose Testing APIs: onNodeWithText, performClick, assertIsDisplayed
- Test recomposition count with ComposeCompiler metrics
- Use Layout Inspector to visualize recomposition regions
- Profile with CPU profiler to find recomposition hotspots