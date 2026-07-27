---
name: game-jam-rapid
description: 
category: creative
tags: [game-jam-rapid]
---

## When to Use
Rapid game prototyping in constrained timeframes (24-72 hour game jams). Use for creativity exercises, team bonding, learning new tools, testing game mechanics quickly, or building portfolio pieces.

## Core Concepts
- **Game jam philosophy**: Ship something playable in the time limit. Scope ruthlessly. Perfect is the enemy of done.
- **Scope control**: The #1 killer of game jams is over-scoping. Design for the time limit, not your ambition.
- **Core loop first**: The single action the player repeats. If the core loop isn't fun, nothing else matters.
- **Theme interpretation**: Jams provide themes. The best entries find unique interpretations, not literal ones.
- **Rapid iteration**: Build → Playtest → Fix → Repeat. 30-minute cycles, not 8-hour marathons.
- **Tool flexibility**: Use whatever you know best. A jam is not the time to learn a new engine.

## Workflow
1. **Theme announcement (Hour 0)**: Brainstorm 10+ concepts in 30 minutes. Pick the one that's most feasible + most interesting.
2. **Core prototype (Hours 1-4)**: Get the core mechanic working. Nothing else matters yet. No art, no menus, just the mechanic.
3. **Game loop (Hours 4-8)**: Add win/lose conditions, progression, and feedback. The game should be "complete" (ugly but functional).
4. **Content & polish (Hours 8-16)**: Add levels, enemies, power-ups, story. Replace placeholder art with final art.
5. **Juice (Hours 16-20)**: Sound effects, particles, screen shake, animations. The details that make it feel alive.
6. **Bug fixing (Hours 20-22)**: Playtest with fresh eyes. Fix game-breaking bugs. Cut features that don't work.
7. **Submission (Hours 22-24)**: Build, test on target platform, write description, submit. Don't cut it close.

## Key Patterns
```markdown
# Core Loop Design
The fundamental question: "What does the player do, and why is it satisfying?"

Examples:
- Tetris: Rotate block → Fits → Line clears → Satisfying sound
- Flappy Bird: Tap → Bird goes up → Avoids pipes → Distance increases
- Vampire Survivors: Move → Auto-attack → Enemies die → XP → Level up → More attacks

# Rapid Prototype Template
# Game: [Name]
# Theme: [Jam theme]
# Interpretation: [How you connect theme to gameplay]
# Core Mechanic: [One sentence — the single action]
# Win Condition: [How do you win?]
# Lose Condition: [How do you lose?]
# Unique Hook: [What makes this different from existing games?]
# Scope: [Exact features you WILL build. Not might.]

# Tech Stack for Speed
Engine: Godot (free, fast iteration) or Unity (larger community)
Art: Pixel art (Aseprite) or simple shapes (programmer art)
Audio: sfxr (retro sounds) or BFXR (modern sounds)
Music: BeepBox (chiptune) or Bosca Ceoil (simple music maker)
Version control: Git with frequent commits

# Time Allocation (24-Hour Jam)
0-1h:   Ideation + core mechanic prototype
1-4h:   Core loop working
4-8h:   Game structure (levels, progression, menus)
8-14h:  Content (art, levels, story)
14-18h: Polish (juice, sound, effects)
18-22h: Bug fixes + playtesting
22-24h: Submission prep + documentation
```

## Pitfalls
- **Over-scoping**: "It'll be an open-world RPG with crafting" in a 48-hour jam. Pick one mechanic. Do it well.
- **Starting with art**: Pretty sprites without a playable core loop = a screensaver, not a game
- **Not playtesting**: Your game makes sense to you because you built it. Fresh eyes reveal everything.
- **Skipping sleep**: Tired developers write bugs. Sleep 4-6 hours. Your output quality doubles.
- **Ignoring the theme**: Judges notice when entries are generic. Connect your game to the theme.
- **Last-minute submission failures**: Build and test your submission process in the first hour. Don't wait until 5 minutes before deadline.

## Verification
- Core loop is fun in the first 10 seconds of play
- Game can be understood without instructions (or with a 1-sentence tutorial)
- Runs at stable frame rate on target platform
- Playtested by 3+ people who didn't build it — did they understand the objective?
- Build submits successfully to the jam platform (itch.io, Global Game Jam site)
- README/description is clear: what the game is, how to play, what makes it special