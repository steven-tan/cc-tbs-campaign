# Fantasy Tactical Battle

Fantasy-themed turn-based tactical combat game (originally Advance Wars-inspired). Single HTML file for now, but structured for future modularization.

## Architecture

The code is organized into five cleanly separated layers in `index.html`:

| Layer | Class | Role |
|---|---|---|
| **Data** | `GameState` | Pure data container — map, units, turn state. Fully serializable. |
| **Logic** | `GameRules` | Static pure functions — movement (BFS), combat (attack + counter), win check. No DOM, no side effects. |
| **AI** | `AIPlayer` | Greedy positional AI. Pure logic — reads state via GameRules, returns action list. No DOM. |
| **Rendering** | `CanvasRenderer` | Reads GameState, draws to canvas. Swappable — just implement `render(state)` and `getCellFromPixel(x,y)`. |
| **Orchestration** | `GameController` | Wires input → rules → state → renderer. Manages start screen, game modes, AI turn execution. |

**Key design rule:** GameState, GameRules, and AIPlayer have zero DOM dependencies. All browser interaction goes through Renderer and Controller.

## Game Modes

- **Start screen** on load: [Play vs AI] [2P Hotseat] [About]
- Play vs AI → player chooses Blue or Red → AI controls the other team
- 2P Hotseat → same as before (local two-player)
- About → game overview overlay, points to in-game Help for detailed stats
- Game-over shows "New Game" button → returns to start screen

## Current Game State

- 9x9 hex grid (pointy-top, odd-r offset coordinates), four terrain types (plains, mountains, forest, water)
- Four unit types: Swordsman (move 3, range 1), Spearman (move 3, range 1), Archer (move 3, range 2), Cavalry (move 4, range 1), 7 per side
- Units rendered as team-colored canvas-path icons (sword, spear+helm, hooded archer+bow, rider on horse) — no circles, icons fill hex space
- Selected unit shown with golden hex border; spent units dimmed via opacity
- Hotseat two-player (Blue/Red) or vs AI, click to select/move/attack
- Units can fire before moving, move then attack, or just do one
- After moving, attack targets auto-highlight if enemies are in range
- Combat has range-gated counter-attacks, damage scales with HP (floors at ~50%)
- HP displayed as actual values (0-100)
- Mountains: +30% defense, 3 move cost for swordsmen/spearmen/archers, impassable for cavalry
- Forest: +30% defense, 2 move cost for all units
- Water: impassable for all units
- Hover tooltips: unit info (name, HP, terrain defense) on any unit; attack preview (damage, flanking, counter, resulting HP) on valid targets
- Mobile long press (~400ms) shows same tooltips as desktop hover
- Click selected unit to deselect in any phase
- Toggleable help panel with unit stats, damage table, terrain modifiers, flanking, combat notes

## AI Player

### Decision Architecture
- `AIPlayer(team, difficulty)` — constructor accepts `'normal'` or `'hard'`
- `planTurn(state)` → clones state, orders units by priority, plans each unit's action sequentially (so later units see updated board)
- `generateCandidateActions()` enumerates all legal combos per unit: pre-move attack + move, move + post-move attack, move only, attack only, do nothing
- `evaluateAction()` scores each candidate; highest score wins
- Hard mode builds a `context` object before planning, passed through to evaluation

### Scoring Factors (both modes)
- **Combat:** damage dealt (+2/HP), kill bonus (+300), focus fire on wounded (+up to 150), type matchup (+30), counter penalty (-1.5/HP), self-destruction (-350)
- **Position:** terrain defense (+1.5/def%), wounded retreat, advance toward enemies when healthy (-3/distance)
- **Tactics:** flanking setup (+25), backstab position (+35), archer range 2 preference (+15)

### Hard Mode Additions
- **Danger map** (`buildDangerMap`): Pre-computes which hexes each enemy can reach+attack next turn using real BFS pathfinding. Stores actual enemy unit references per hex for precise damage calculation.
- **Coordinated focus fire** (`identifyFocusTargets`): Scans board for killable enemies (total reachable damage >= HP). Top 2 focus targets get +100 attack bonus, causing multiple units to pile onto the same target.
- **Kill-danger avoidance**: If total incoming damage at a hex >= unit's HP, -250 penalty. Prevents walking into positions where the unit can be killed next turn.
- **Archer safety**: Archers get -25 per threat at their hex (on top of general threat penalty). Also -30 (vs -10 normal) for standing adjacent to enemies.
- **Earlier retreat**: Wounded threshold raised from 40 to 50 HP, retreat pull increased (+10/distance vs +8).

### Unit Priority Order
Kill-capable first (+500), archers before melee (+100), adjacent before distant (+80/+40), strong matchups (+30), wounded last (-50)

### AI Turn Visualization
- Staggered delays: select (400ms) → highlight (300ms) → execute (500ms) → pause (200ms)
- Space or Esc to skip animation
- Info panel shows "(Space/Esc to skip)" during AI turn
- Player input blocked during AI turn

## Combat Mechanics

- **Damage formula:** `base * atkMod * (100 - defBonus) / 100`, minimum 1
- **Damage scaling:** `atkMod = 0.5 + 0.5 * (hp / 100)` — wounded units deal 50-100% damage
- **Type advantages:** Swordsman strong vs Spearman; Spearman & Archer strong vs Cavalry; Cavalry strong vs Swordsman & Archer. Strong = 70 base damage, neutral = 55.
- **Counter-attacks:** Defender returns fire using their reduced (post-damage) HP, but only if their range reaches the attacker. Melee units can't counter archers shooting from 2 hexes away.
- **Terrain defense:** Mountains and forests give +30% damage reduction to defender
- **Flanking (melee only):** Attacker + 1 ally adjacent to defender = +10% attack. Attacker + 2+ allies = +20%. Ally directly behind defender (backstab) = +25%, overrides count-based bonuses. Counter-attacks do not get flanking bonuses.

## Hex Grid Details

- **Coordinate system:** Odd-r offset (odd rows shifted right). Map is still a 2D array, units store `{row, col}`.
- **Key functions:** `hexNeighbors(row, col)`, `hexToPixel(row, col)`, `pixelToHex(px, py)`, `hexDistance(r1, c1, r2, c2)` — all at module level, shared by rules and renderer.
- **Neighbor lookup:** Uses `HEX_DIRS[row & 1]` — 6 direction offsets that depend on row parity.
- **Range checking:** `hexDistance()` converts to cube coordinates for accurate hex distance.

## Data-Driven Design

Game balance lives in constants at the top of the file: `UNIT_DEFS`, `TERRAIN_DEFS`, `DAMAGE_TABLE`, `FLANKING`. Tune these without touching logic. Adding new unit/terrain types means adding entries to these tables.

## Planned Extension Points

- **Save/load:** `state.serialize()` / `GameState.deserialize()` already exist
- **Multiplayer (roadmap):** Lightweight 2-player online via shared link (no accounts/passwords). Two candidate approaches: Firebase Realtime DB (reliable, free tier, client-side SDK) or PeerJS/WebRTC (peer-to-peer, no external project). Room codes shared out-of-band (e.g. Discord). Deferred until after AI and core mechanics stabilize.
- **AI difficulty tiers:** Current AI is tuned to be challenging. Difficulty levels can be added by adjusting scoring weights.
- **New renderer:** Swap CanvasRenderer for WebGL/DOM/terminal
- **Map generation:** Random terrain placement (logic layer, not rendering)
- **More terrain/unit types:** Add entries to TERRAIN_DEFS, UNIT_DEFS, DAMAGE_TABLE (forest, water, cavalry already added)
