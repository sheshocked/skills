---
name: procedural-generation
description: - Creating variety in game worlds without manual authoring
category: threed
tags: [procedural-generation]
---

## When to Use
- Generating levels, terrain, dungeons, or cities algorithmically
- Creating variety in game worlds without manual authoring
- Roguelike/roguelite games with random layouts
- Simulation: ecosystem generation, city growth
- Data-driven content pipelines for large game worlds

## Core Concepts
- Wave Function Collapse (WFC): constraint-based tile placement
- Perlin/Simplex noise: smooth gradient-based terrain generation
- Voronoi diagrams: natural-looking partitioning for regions
- L-systems: grammar-based plant/tree generation
- Cellular automata: cave/dungeon generation rules
- Seeded randomness: reproducible results with same seed
- Rule-based generation: grammars, Markov chains, Bézier curves

## Workflow
1. Choose algorithm based on content type (noise for terrain, WFC for tiles)
2. Set up seed input for reproducibility
3. Generate base layer (noise map, grid, or graph)
4. Apply post-processing (smoothing, erosion, cleanup)
5. Add variation layers (vegetation placement, detail objects)
6. Test with multiple seeds for edge cases
7. Cache generated results to disk for reuse

## Key Patterns
```python
# Perlin noise terrain generation
import numpy as np
from noise import pnoise2

def generate_heightmap(width, height, scale=100.0, octaves=6, seed=42):
    heightmap = np.zeros((width, height))
    for x in range(width):
        for y in range(height):
            heightmap[x][y] = pnoise2(
                x / scale + seed,
                y / scale + seed,
                octaves=octaves,
                persistence=0.5,
                lacunarity=2.0
            )
    return np.clip(heightmap, -1, 1)

# Convert to terrain types
def classify_terrain(heightmap):
    types = np.zeros_like(heightmap, dtype=int)
    types[heightmap < -0.3] = 0  # deep water
    types[(heightmap >= -0.3) & (heightmap < -0.1)] = 1  # shallow water
    types[(heightmap >= -0.1) & (heightmap < 0.1)] = 2   # sand/beach
    types[(heightmap >= 0.1) & (heightmap < 0.5)] = 3    # grass
    types[(heightmap >= 0.5) & (heightmap < 0.8)] = 4    # rock
    types[heightmap >= 0.8] = 5                            # snow
    return types
```

```python
# Simple dungeon generator (room + corridor)
import random

def generate_dungeon(width, height, room_count=10, seed=42):
    random.seed(seed)
    rooms = []
    grid = np.zeros((width, height), dtype=int)  # 0=wall, 1=floor

    for _ in range(room_count):
        rw = random.randint(4, 10)
        rh = random.randint(4, 10)
        rx = random.randint(1, width - rw - 1)
        ry = random.randint(1, height - rh - 1)
        rooms.append((rx, ry, rw, rh))
        grid[rx:rx+rw, ry:ry+rh] = 1

    # Connect rooms with corridors
    for i in range(len(rooms) - 1):
        x1, y1 = rooms[i][0] + rooms[i][2]//2, rooms[i][1] + rooms[i][3]//2
        x2, y2 = rooms[i+1][0] + rooms[i+1][2]//2, rooms[i+1][1] + rooms[i+1][3]//2
        for x in range(min(x1, x2), max(x1, x2) + 1):
            grid[x, y1] = 1
        for y in range(min(y1, y2), max(y1, y2) + 1):
            grid[x2, y] = 1
    return grid
```

## Pitfalls
- Unseeded randomness makes debugging impossible — always use seeds
- WFC can fail with unsatisfiable constraints — implement backtracking
- Noise frequency too high: visual noise; too low: featureless terrain
- Generated content needs manual QA — not everything is algorithm-friendly
- Performance: expensive generation should be done in background threads

## Verification
- Generate 100 seeds and check for visual diversity
- Playtest generated levels for navigability (no dead ends in key areas)
- Validate WFC tile adjacency constraints programmatically
- Measure generation time: < 1 second for player-visible content
- Compare with hand-crafted benchmarks for quality targets