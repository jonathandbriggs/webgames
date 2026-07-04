# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of interactive puzzle games delivered as standalone HTML files. Each game is a self-contained file with embedded CSS and JavaScript—no build process, no dependencies, no server required. Games are designed for mobile and desktop, with localStorage used for persistent state (e.g., level progression).

## Architecture & Key Patterns

### Game Structure

Each game follows a consistent pattern:

1. **State Management**: Game state lives in module-scope variables (`level`, `lives`, `snakes`, `arrows`, etc.). State changes trigger redraws/reflows.
2. **Grid-Based Mechanics**: Both existing games use a 2D grid where:
   - Cells are indexed as `key = y * cols + x` (or similar)
   - Movement/rays cast from one grid cell in cardinal directions
   - Solvability is guaranteed by building in reverse removal order (see "Generation" below)
3. **Pointer Input**: Tap/click handling uses pointer events:
   - **Amaze GO** (amaze-go.html): `pointerdown` on the canvas, retrieves grid position and calls `tap(snake)`
   - **Arrow Maze** (arrow-maze.html): `pointerup` on individual tiles, calls `tap(arrow)`
   - Both should use consistent responsiveness (`touch-action:manipulation`, `-webkit-tap-highlight-color:transparent`)
4. **Animation & Rendering**:
   - **Amaze GO**: Canvas-based with requestAnimationFrame, snakes slide along a polyline track during exit and bump animations
   - **Arrow Maze**: DOM-based with Web Animations API, tiles scale/fade/translate during fly and bounce animations
5. **Procedural Generation**: Both games generate solvable puzzles with deterministic seeding (if needed) and attempt strategies to maximize board density

### Key Game Mechanics

**Amaze GO** (`amaze-go.html`, ~450 LOC):
- Player taps snakes to remove them; snakes slide out if their forward ray is clear
- If forward ray is blocked, the snake bumps backward
- 3 hearts, lose one per bump, win when all snakes are removed
- Level progression via localStorage; harder levels have larger grids and more snakes
- Colors assigned via graph coloring so touching snakes never share a color

**Arrow Maze** (`arrow-maze.html`, ~370 LOC):
- Player taps arrows to send them flying in their direction
- Arrows fly off the board if unobstructed; bounce back if blocked
- Shape changes each game (circle, diamond, heart, star, plus, hexagon, crescent, blob)
- Solvable direction assignment (reverse removal order guarantees all orderings are valid)
- Confetti on win; staggered pop-in animation on game start

### Layout & Scaling

Both games use fluid layouts:
- Canvas or board container sized to fill viewport minus header/margins
- `cell` size computed per window size and grid dimensions
- `layout()` called on resize and after generation/state change
- High-DPI support via `devicePixelRatio` (Amaze) or handled by browser (Arrow Maze)

## Development Notes

**How to develop/test**: Open any `.html` file in a browser. Changes are live—just refresh. No build step, no dev server needed.

**Consistency across games**:
- Both use dark backgrounds with light text
- Both disable default tap highlight and long-press menu
- Both use `touch-action:manipulation` for responsive input
- Keep pointer event handling consistent: standardize on which event fires the tap (currently mixed: Amaze uses `pointerdown`, Arrow uses `pointerup`)
- When adding new games, follow the same folder structure (self-contained `.html` file in `public/`, linked from `public/index.html`)

**State Persistence**: Use `localStorage` to save progress (e.g., `localStorage.getItem('amazeLevel')`). Always namespace keys to avoid collisions.

**Performance**: Canvas drawing happens every frame in Amaze; DOM animations in Arrow Maze are GPU-accelerated. Avoid adding heavy computations inside `tick()` or animation callbacks.

## Common Tasks

- **Add a new game**: Create `public/game-name.html` as a self-contained file, add a link in `public/index.html`
- **Tweak colors/sizes**: Edit the `<style>` block at the top of each file
- **Change difficulty/generation**: Adjust the `PALETTE`, `DIRS`, or generation parameters in the `<script>` section
- **Fix tap responsiveness**: Check pointer event listener (which event type, event.preventDefault/stopPropagation if needed) and confirm `touch-action` is set to `manipulation`
