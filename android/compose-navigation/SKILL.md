---
name: compose-navigation
description: Establish type-safe navigation graphs and customized screen transition animations in Compose.
category: android
tags: [compose-navigation, type-safety, transitions, kotlin]
---

# Compose Navigation

## When to Use
Use when implementing modular navigations with custom sliding/fading screen transitions.

## Prerequisites
- Compose Navigation dependency `androidx.navigation:navigation-compose`.

## Workflow
1. Define type-safe screens as Serializable classes or objects.
2. Build the `NavHost` containing all screen paths.
3. Configure `enterTransition` and `exitTransition` animations.

## Key Patterns
```kotlin
@Serializable
sealed class Screen {
    @Serializable data object Dashboard : Screen()
    @Serializable data class Settings(val theme: String) : Screen()
}

@Composable
fun AppNavigation(navController: NavHostController) {
    NavHost(
        navController = navController,
        startDestination = Screen.Dashboard,
        enterTransition = { slideInHorizontally(initialOffsetX = { it }) },
        exitTransition = { slideOutHorizontally(targetOffsetX = { -it }) }
    ) {
        composable<Screen.Dashboard> { DashboardScreen(navController) }
        composable<Screen.Settings> { backStackEntry ->
            val settings = backStackEntry.toRoute<Screen.Settings>()
            SettingsScreen(settings.theme)
        }
    }
}
```

## Pitfalls
- **State losses on orientation change:** Pass only primitive parameters in routes; fetch complex structures from shared ViewModels.
- **Deep nesting paths:** Keep screen routes flat to avoid navigation graph pollution.

## Verification
- Verify argument parsing works when pushing deep links.
- Test navigation actions do not leak instances of ViewModels.
