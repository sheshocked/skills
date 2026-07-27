---
name: android-localization
description: 
category: android
tags: [android-localization]
---

## When to Use
Use this skill when localizing Android apps: RTL support, Persian/Arabic text, plural rules, per-app language.

## Key Patterns
```xml
<!-- strings.xml (English) -->
<string name="greeting">Hello, %1$s!</string>
<string name="items_count">%d items</string>
<string-array name="languages">
    <item>English</item>
</string-array>

<!-- strings.xml (Persian) -->
<string name="greeting">سلام %1$s!</string>
<string name="items_count">%d آیتم</string>
```

```kotlin
// Per-app language (Android 13+)
AppCompatDelegate.setApplicationLocales(
    LocaleListCompat.forLanguageTags("fa")
)

// RTL support
android:supportsRtl="true"
// Use start/end instead of left/right in layouts
```

## Pitfalls
- **RTL layout**: Test mirrored UI for RTL languages
- **Font scaling**: Persian/Arabic text may need larger font sizes
- **Plural rules**: Different languages have different plural forms

## Verification
- Test with RTL language (Persian/Arabic)
- Verify text doesn't overflow containers
- Check per-app language setting works