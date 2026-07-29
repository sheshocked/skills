---
name: compose-horizontal-scroll
description: Build premium horizontal scrolling wrappers in Jetpack Compose to prevent vertical layouts compression.
category: android
tags: [horizontal-scroll, compose-layout, recycler-compose, kotlin]
---

# Compose Horizontal Scroll

## When to Use
Use when rendering segmented lists (e.g. server profiles, config plans) where vertical compression makes UI unreadable on small screens.

## Prerequisites
- Jetpack Compose UI dependencies.

## Workflow
1. Set up a `LazyRow` with structured spacing.
2. Bind horizontal paging behaviors with snappers if carousel styles are requested.
3. Implement item layout dynamically scaling according to view constraints.

## Key Patterns
```kotlin
@Composable
fun ProfileHorizontalScroll(profiles: List<VpnProfile>, activeId: String, onSelect: (String) -> Unit) {
    LazyRow(
        modifier = Modifier.fillMaxWidth(),
        contentPadding = PaddingValues(horizontal = 16.dp, vertical = 8.dp),
        horizontalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        items(profiles, key = { it.id }) { profile ->
            ProfileCard(
                profile = profile,
                isActive = profile.id == activeId,
                onClick = { onSelect(profile.id) }
            )
        }
    }
}
```

## Pitfalls
- **Excessive item heights:** Inside horizontal scrolls, items must specify static bounds to avoid viewport overflow.
- **Missing item keys:** Fails to animate states without unique ID mapping.

## Verification
- Test rendering with 50+ elements; scroll horizontally to verify layout stability.
- Measure FPS during scrolling to confirm zero frame drops.
