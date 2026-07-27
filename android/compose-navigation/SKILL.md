---
name: compose-navigation
description: 
category: android
tags: [compose-navigation]
---

## When to Use
Use this skill when implementing Compose Navigation: type-safe navigation, nested graphs, deep links, predictive back.

## Core Concepts
- **NavHost**: Container for composable destinations
- **NavController**: Navigate between destinations
- **Type-safe args**: Use @Serializable data classes for arguments
- **Nested graphs**: Group related routes
- **Deep links**: Open specific screens from URLs

## Key Patterns
```kotlin
// Route definitions
@Serializable object Home
@Serializable data class Profile(val userId: Long)
@Serializable object Settings

// NavHost
NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen(onNavigateToProfile = { navController.navigate(Profile(userId = it)) }) }
    composable<Profile> { backStackEntry ->
        val profile = backStackEntry.toRoute<Profile>()
        ProfileScreen(userId = profile.userId)
    }
    composable<Settings> { SettingsScreen() }
}

// Deep links
composable<Home>(
    deepLinks = listOf(navDeepLink<Home>(basePath = "myapp://home"))
) { ... }

// Nested navigation
navigation(startDestination = "list", route = "profile") {
    composable("list") { ProfileList() }
    composable("detail/{id}") { ... }
}
```

## Pitfalls
- **Argument types**: Only primitive types and @Serializable classes
- **Nested back stack**: PopUpTo and restoreState for bottom nav
- **Predictive back**: Test with Android 14+ back gesture
- **State restoration**: Use rememberSaveable with navigation args

## Verification
- Test deep links with adb shell am start
- Verify back navigation behavior
- Test state preservation across config changes
- Check nested graph navigation