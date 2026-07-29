---
name: compose-horizontal-scroll
description: Design optimized horizontal scrolling server/profile carousels in Compose to bypass vertical compression limits on compact viewports.
category: android
tags: [compose, jetpack-compose, horizontal-scroll, lazyrow, carousels, responsive-ui]
---

# Jetpack Compose Horizontal Scroll wrapper Masterclass

## When to Use
Use when rendering lists of connection cards, server profiles, or tariff configurations in Android VPN UIs. Standard vertical lists compress card elements, creating layout bloat on mobile screens. Wrapping them in structured horizontal carousels keeps interfaces clean and readable.

## Prerequisites
- Jetpack Compose Layout libraries configured.

## Workflow
1. Set up a `LazyRow` container with content paddings.
2. Bind selected states with animated borders and scale parameters.
3. Configure layout snapping using `rememberFlingBehavior` to lock items in position on swipe.

## Key Patterns

### Compose Card Carousel (ProfileCarousel.kt)
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
            
            // Premium scale transitions on active cards
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
- **Missing items keys:** If `items()` lacks a key mapper, Compose redraws all cards during sorting changes, dropping frames. Always map `key = { it.id }`.
- **Clipping shadow artifacts:** Outer shadows get cut off by `LazyRow` bounds. Enforce `clipToPadding = false` on parent components.

## Verification
- Run UI test scrolling through 100 profiles to verify recomposition limits.
- Inspect layouts to verify correct dimensions across small screens.
