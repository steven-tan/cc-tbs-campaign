# Fantasy Tactical Battle

Fantasy-themed turn-based tactical combat game (originally Advance Wars-inspired). Single HTML file for now, but structured for future modularization.

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
- Two unit types: Infantry (I, range 1) and Archer (A, range 2), 6 per side
- Hotseat two-player (Blue/Red), click to select/move/attack
- Units can fire before moving, move then attack, or just do one
- After moving, attack targets auto-highlight if enemies are in range
- Combat has range-gated counter-attacks, damage scales with HP (floors at ~50%)
- HP displayed as actual values (0-100)
- Mountains: +30% defense, 2 move cost for infantry and archers
- Toggleable help panel with unit stats, damage table, terrain modifiers, combat notes

## Combat Mechanics

- **Damage formula:** `base * atkMod * (100 - defBonus) / 100`, minimum 1
- **Damage scaling:** `atkMod = 0.5 + 0.5 * (hp / 100)` — wounded units deal 50-100% damage
- **Counter-attacks:** Defender returns fire using their reduced (post-damage) HP, but only if their range reaches the attacker. Infantry can't counter archers shooting from 2 hexes away.
- **Terrain defense:** Mountains give +30% damage reduction to defender

## Hex Grid Details

- **Coordinate system:** Odd-r offset (odd rows shifted right). Map is still a 2D array, units store `{row, col}`.
- **Key functions:** `hexNeighbors(row, col)`, `hexToPixel(row, col)`, `pixelToHex(px, py)`, `hexDistance(r1, c1, r2, c2)` — all at module level, shared by rules and renderer.
- **Neighbor lookup:** Uses `HEX_DIRS[row & 1]` — 6 direction offsets that depend on row parity.
- **Range checking:** `hexDistance()` converts to cube coordinates for accurate hex distance.

## Data-Driven Design

Game balance lives in constants at the top of the file: `UNIT_DEFS`, `TERRAIN_DEFS`, `DAMAGE_TABLE`. Tune these without touching logic. Adding new unit/terrain types means adding entries to these tables.

## Planned Extension Points (not yet implemented)

- **Save/load:** `state.serialize()` / `GameState.deserialize()` already exist
- **AI player:** Hook into GameController — check currentTeam, call AI instead of waiting for clicks
- **New renderer:** Swap CanvasRenderer for WebGL/DOM/terminal
- **Networking:** Sync serialized GameState between clients
- **Map generation:** Random terrain placement (logic layer, not rendering)
- **More terrain/unit types:** Add entries to TERRAIN_DEFS, UNIT_DEFS, DAMAGE_TABLE
