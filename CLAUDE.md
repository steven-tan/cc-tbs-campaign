# Tactical Battle System (TBS)

Advance Wars-inspired turn-based tactical combat game. Single HTML file for now, but structured for future modularization.

## Architecture

The code is organized into four cleanly separated layers in `index.html`:

| Layer | Class | Role |
|---|---|---|
| **Data** | `GameState` | Pure data container — map, units, turn state. Fully serializable. |
| **Logic** | `GameRules` | Static pure functions — movement (BFS), combat (attack + counter), win check. No DOM, no side effects. |
| **Rendering** | `CanvasRenderer` | Reads GameState, draws to canvas. Swappable — just implement `render(state)` and `getCellFromPixel(x,y)`. |
| **Orchestration** | `GameController` | Wires input → rules → state → renderer. Entry point for AI, networking, save/load. |

**Key design rule:** GameState and GameRules have zero DOM dependencies. All browser interaction goes through Renderer and Controller.

## Current Game State

- 8x8 hex grid (pointy-top, odd-r offset coordinates), two terrain types (plains, mountains)
- Two unit types: Infantry (I) and Tank (T), 6 per side
- Hotseat two-player (Blue/Red), click to select/move/attack
- 6-direction movement and adjacency
- Combat has counter-attacks, damage scales with HP
- Mountains: +30% defense, 2 move cost for infantry, impassable for tanks

## Hex Grid Details

- **Coordinate system:** Odd-r offset (odd rows shifted right). Map is still a 2D array, units store `{row, col}`.
- **Key functions:** `hexNeighbors(row, col)`, `hexToPixel(row, col)`, `pixelToHex(px, py)` — all at module level, shared by rules and renderer.
- **Neighbor lookup:** Uses `HEX_DIRS[row & 1]` — 6 direction offsets that depend on row parity.
- **Pixel conversion:** Goes through axial coordinates with cube rounding for accurate click detection.

## Data-Driven Design

Game balance lives in constants at the top of the file: `UNIT_DEFS`, `TERRAIN_DEFS`, `DAMAGE_TABLE`. Tune these without touching logic.

## Planned Extension Points (not yet implemented)

- **Save/load:** `state.serialize()` / `GameState.deserialize()` already exist
- **AI player:** Hook into GameController — check currentTeam, call AI instead of waiting for clicks
- **New renderer:** Swap CanvasRenderer for WebGL/DOM/terminal
- **Networking:** Sync serialized GameState between clients
- **More terrain/unit types:** Add entries to TERRAIN_DEFS, UNIT_DEFS, DAMAGE_TABLE
