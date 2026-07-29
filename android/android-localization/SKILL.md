---
name: android-localization
description: Configure RTL mirroring layouts, typography, and resources for Arabic and Persian languages.
category: android
tags: [rtl, persian-ui, localization, compose-rtl, kotlin]
---

# Android Localization

## When to Use
Use when localizing Android apps for Persian/Arabic users, ensuring layout mirrors correctly and native numbers format properly.

## Prerequisites
- Set `android:supportsRtl="true"` in AndroidManifest.xml.

## Workflow
1. Use start/end constraints instead of left/right margins.
2. Mirror navigation drawer direction.
3. Apply correct Persian/Arabic typography configurations.

## Key Patterns
```kotlin
// Compose RTL support is automatic when using local configurations:
CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl) {
    Row(
        modifier = Modifier.fillMaxWidth().padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text("پروفایل‌ها") // Appears on the right
        Icon(Icons.Default.ArrowBack, contentDescription = null) // Automatically mirrored
    }
}
```

## Pitfalls
- **Static icons:** Icons containing direction (arrows) must specify autoMirror: `painterResource(id = R.drawable.ic_arrow).apply { autoMirror = true }`.
- **Text alignment conflicts:** Static "left" alignment looks broken in RTL layouts. Use `TextAlign.Start` instead of `TextAlign.Left`.

## Verification
- Toggle device language to Persian or toggle RTL mode in developer settings.
- Verify component flow alignment visually.
