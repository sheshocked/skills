---
name: android-widgets-glance
description: 
category: android
tags: [android-widgets-glance]
---

## When to Use
Use this skill when building home screen widgets with Jetpack Glance.

## Key Patterns
```kotlin
class MyWidget : GlanceAppWidget() {
    override suspend fun provideGlance(context: Context, id: GlanceId) {
        provideContent {
            GlanceTheme {
                Column {
                    Text("Status: Connected")
                    Button(onClick = actionRunCallback<ToggleAction>()) {
                        Text("Toggle")
                    }
                }
            }
        }
    }
}

// Widget receiver
class MyWidgetReceiver : GlanceAppWidgetReceiver() {
    override val glanceAppWidget = MyWidget()
}
```

## Pitfalls
- **Update frequency**: Widgets update every 30 min minimum
- **RemoteViews**: Glance builds on RemoteViews — limited interactivity
- **Size classes**: Test on all widget sizes (1x1 to 4x4)

## Verification
- Test on home screen with different sizes
- Verify state updates correctly
- Check battery impact