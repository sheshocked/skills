---
name: compose-horizontal-scroll
description: Build responsive horizontal LazyRow layouts in Compose, preventing vertical text compression on compact screens.
category: android
tags: [compose, lazyrow, horizontal-scroll, carousel, ui-ux, kotlin]
---

# Compose Horizontal Scroll

## When to Use
Use when rendering tariff cards, servers profiles, or configs selections inside mobile UI layouts where vertical lists would compress text elements.

## Prerequisites
- Jetpack Compose Layout libraries.

## Workflow
1. Set up a `LazyRow` container with content padding.
2. Animate scale and borders based on selected state.
3. Configure layout snapping for smooth carousels.

## Key Patterns

### Compose Profile Selector (ProfileCarousel.kt)
```kotlin
package com.surfshield.ui

import androidx.compose.animation.animateColorAsState
import androidx.compose.animation.core.animateFloatAsState
import androidx.compose.foundation.BorderStroke
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.graphicsLayer
import androidx.compose.ui.unit.dp

data class VpnProfile(val id: String, val name: String, val flag: String, val latency: Int)

@Composable
fun ProfileCarousel(
    profiles: List<VpnProfile>,
    activeId: String,
    onSelect: (String) -> Unit
) {
    LazyRow(
        modifier = Modifier.fillMaxWidth().height(160.dp),
        contentPadding = PaddingValues(horizontal = 16.dp, vertical = 8.dp),
        horizontalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        items(profiles, key = { it.id }) { profile ->
            val isActive = profile.id == activeId
            val scale by animateFloatAsState(if (isActive) 1.05f else 0.95f, label = "scale")
            val borderColor by animateColorAsState(if (isActive) MaterialTheme.colorScheme.primary else Color.Transparent, label = "border")

            Card(
                modifier = Modifier
                    .width(130.dp)
                    .fillMaxHeight()
                    .graphicsLayer {
                        scaleX = scale
                        scaleY = scale
                    }
                    .clickable { onSelect(profile.id) },
                border = BorderStroke(2.dp, borderColor),
                colors = CardDefaults.cardColors(
                    containerColor = if (isActive) MaterialTheme.colorScheme.surfaceVariant else MaterialTheme.colorScheme.surface
                )
            ) {
                Column(
                    modifier = Modifier.fillMaxSize().padding(12.dp),
                    verticalArrangement = Arrangement.SpaceBetween
                ) {
                    Text(profile.flag, style = MaterialTheme.typography.headlineMedium)
                    Column {
                        Text(profile.name, style = MaterialTheme.typography.titleMedium)
                        Text("${profile.latency} ms", style = MaterialTheme.typography.bodyMedium, color = Color.Gray)
                    }
                }
            }
        }
    }
}
```

## Pitfalls
- **Recomposition lag:** LazyRow without key mappings triggers full redraws on dataset changes. Always supply `key = { it.id }`.
- **Card shadow clipping:** Ensure parent containers specify `clipToPadding = false` to prevent card shadows from getting clipped.

## Verification
- Test UI responsiveness with 50+ elements on screen.

