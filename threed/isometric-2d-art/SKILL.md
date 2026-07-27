---
name: isometric-2d-art
description: - Isometric tilesets for level editors and map builders
category: threed
tags: [isometric-2d-art]
---

## When to Use
- Creating isometric game art (2.5D perspective for strategy/RPG games)
- Isometric tilesets for level editors and map builders
- Infographics and technical illustrations with isometric view
- Isometric UI elements and HUD design
- Pixel art isometric environments

## Core Concepts
- Isometric projection: 30° angles, no perspective distortion
- Tile grid: diamond-shaped tiles (2:1 ratio width:height)
- Standard isometric tile: 64×32 pixels (2:1) or 128×64 for high-res
- Stacking rules: objects on tiles must respect layering order
- Pixel-perfect alignment: even pixel dimensions prevent sub-pixel artifacts
- Light direction: consistent lighting across all tiles (typically top-left)
- Color ramp: limited palette for consistent style

## Workflow
```
# Tileset creation pipeline
1. Define grid size: 64×32 for standard, 128×64 for hi-res
2. Create base terrain tiles: grass, water, stone, sand
3. Add edge/transition tiles for terrain blending
4. Create props: trees, buildings, rocks (fit within tile bounds)
5. Add shadows consistent across all tiles
6. Export as tileset PNG or individual tiles
7. Import into tile editor (Tiled, LDtk, or custom)
```

## Key Patterns
```python
# Generate isometric grid coordinates
def iso_to_pixel(tile_x, tile_y, tile_width=64, tile_height=32):
    # Convert tile coords to pixel coords (top-left of tile)
    pixel_x = (tile_x - tile_y) * (tile_width // 2)
    pixel_y = (tile_x + tile_y) * (tile_height // 2)
    return pixel_x, pixel_y

def pixel_to_iso(pixel_x, pixel_y, tile_width=64, tile_height=32):
    # Convert pixel coords to tile coords
    tile_x = (pixel_x / (tile_width // 2) + pixel_y / (tile_height // 2)) / 2
    tile_y = (pixel_y / (tile_height // 2) - pixel_x / (tile_width // 2)) / 2
    return round(tile_x), round(tile_y)

# A* pathfinding on isometric grid
import heapq

def astar_isometric(start, goal, walkable):
    # A* on isometric grid with 4-directional movement
    open_set = [(0, start)]
    came_from = {}
    g_score = {start: 0}
    
    while open_set:
        _, current = heapq.heappop(open_set)
        if current == goal:
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            return path[::-1]
        
        for dx, dy in [(-1,0),(1,0),(0,-1),(0,1)]:
            neighbor = (current[0]+dx, current[1]+dy)
            if neighbor not in walkable:
                continue
            tentative = g_score[current] + 1
            if tentative < g_score.get(neighbor, float('inf')):
                came_from[neighbor] = current
                g_score[neighbor] = tentative
                f = tentative + abs(neighbor[0]-goal[0]) + abs(neighbor[1]-goal[1])
                heapq.heappush(open_set, (f, neighbor))
    return []
```

Isometric sprite rendering:
```javascript
// HTML5 Canvas isometric renderer
class IsometricRenderer {
  constructor(ctx, tileW = 64, tileH = 32) {
    this.ctx = ctx;
    this.tileW = tileW;
    this.tileH = tileH;
  }

  drawTile(x, y, sprite, elevation = 0) {
    const screenX = (x - y) * (this.tileW / 2);
    const screenY = (x + y) * (this.tileH / 2) - elevation * this.tileH;
    this.ctx.drawImage(sprite, screenX - this.tileW/2, screenY - sprite.height);
  }

  // Sort order: draw back-to-front
  renderMap(map) {
    for (let y = 0; y < map.height; y++) {
      for (let x = 0; x < map.width; x++) {
        const tile = map.get(x, y);
        if (tile) this.drawTile(x, y, tile.sprite, tile.elevation);
      }
    }
  }
}
```

## Pitfalls
- Sub-pixel artifacts: always use even tile dimensions (64×32, not 63×31)
- Z-ordering errors: objects must be sorted by (x+y) draw order
- Shadow consistency: all shadows must use same angle and length
- Tile seams: 1px gaps appear if tiles aren't pixel-aligned
- Too many unique tiles: limit palette, reuse tiles with color variation
- Animation on isometric: rotating sprites breaks isometric illusion — use frame-based

## Verification
- Zoom in 400%: verify pixel-perfect alignment, no sub-pixel gaps
- Render 1000+ tiles: check for visual consistency and performance
- Test pathfinding: characters walk correctly without clipping through tiles
- Check tile transitions: terrain blending looks natural at boundaries
- Export and import into Tiled/LDtk: verify tile IDs and layer structure