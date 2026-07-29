---
name: dark-mode-dynamic
description: Construct dynamic theme providers, pure OLED black variables, and local persistence theme configurations.
category: design
tags: [dark-mode, oled-black, tailwind, compose, theme-persistence]
---

# Dynamic Dark Mode & OLED Black Masterclass

## When to Use
Use when designing dark aesthetics interfaces (like Incy theme structures) that transition between standard gray-dark and pure battery-saving OLED black (`#000000`) based on device configurations.

## Prerequisites
- Tailwind CSS or Jetpack Compose UI configurations.

## Workflow
1. Configure primary, surface, and window color maps.
2. Bind local theme state dynamically using contexts or local storage values.
3. Apply transitions preventing visual flickering on load.

## Key Patterns

### Compose OLED Dark Theme (Theme.kt)
```kotlin
package com.surfshield.ui.theme

import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.runtime.Composable
import androidx.compose.ui.graphics.Color

private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF2563EB),
    background = Color(0xFFF8FAFC),
    surface = Color(0xFFFFFFFF)
)

private val StandardDarkColorScheme = darkColorScheme(
    primary = Color(0xFF3B82F6),
    background = Color(0xFF0F172A), // Slate Gray
    surface = Color(0xFF1E293B)
)

private val OledDarkColorScheme = darkColorScheme(
    primary = Color(0xFF3B82F6),
    background = Color(0xFF000000), // Pure OLED Black
    surface = Color(0xFF0D0D0D),   // Off-black cards
    surfaceVariant = Color(0xFF1A1A1A)
)

@Composable
fun SurfShieldTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    useOledBlack: Boolean = true, // Enable OLED option
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        darkTheme && useOledBlack -> OledDarkColorScheme
        darkTheme -> StandardDarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

## Pitfalls
- **Transition flashbangs:** Initial HTML rendering defaults to light mode before Javascript parses user selections. Inject inline script tags inside `<head>` resolving settings instantly.
- **Low Contrast Details:** Pure black backgrounds require higher contrast borders (`#1a1a1a` or `#333333`) to prevent elements merging visually.

## Verification
- Test UI under varying brightness environments.
- Verify battery utilization curves on OLED hardware screens.
