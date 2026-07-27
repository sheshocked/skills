---
name: material-design-3
description: 
category: android
tags: [material-design-3]
---

## When to Use
Use this skill when implementing Material Design 3 theming: dynamic color, custom color schemes, typography, dark/AMOLED themes.

## Key Patterns
```kotlin
// Theme setup
@Composable
fun SurfShieldTheme(
    themeMode: ThemeMode = ThemeMode.SYSTEM,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        themeMode == ThemeMode.AMOLED -> darkColorScheme(
            background = Color.Black,
            surface = Color.Black,
            surfaceVariant = Color(0xFF1A1A1A)
        )
        themeMode == ThemeMode.DARK -> darkColorScheme()
        else -> lightColorScheme()
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}

// Dynamic color (Android 12+)
if (Build.VERSION.SDK_INT >= 31) {
    val dynamicColor = dynamicDarkColorScheme(context)
    MaterialTheme(colorScheme = dynamicColor) { ... }
}

// Custom typography
val Typography = Typography(
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Bold,
        fontSize = 32.sp
    )
)
```

## Pitfalls
- **Dynamic color**: Only available on Android 12+ — provide fallback
- **AMOLED black**: Use pure #000000, not dark gray
- **Typography scale**: Follow M3 type scale (display/headline/title/body/label)

## Verification
- Test light, dark, and AMOLED modes
- Verify dynamic color on Android 12+ device
- Check contrast ratios with accessibility scanner
- Test font scaling (accessibility settings)