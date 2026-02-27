# Fantasy Tactical Battle

A turn-based tactical combat game on a hex grid, inspired by Advance Wars. Play against an AI opponent or a friend in hotseat mode.

**[Play it here](https://steven-tan.github.io/cc-tbs-initial/)**

## How to Play

- Choose **Play vs AI** (Normal or Hard difficulty, pick your team) or **2P Hotseat**
- **Select** a unit by clicking it on your turn (Blue goes first)
- **Move** by clicking a highlighted hex
- **Attack** enemies highlighted in red — you can fire before or after moving
- **End Turn** when you're done with all your units
- Eliminate all enemy units to win
- Hover over units for info, hover over valid targets for damage previews
- On mobile, long press for tooltips

## Units

| Unit | Move | Range | Strong vs |
|------|------|-------|-----------|
| Swordsman | 3 | 1 | Spearman |
| Spearman | 3 | 1 | Cavalry |
| Archer | 3 | 2 | Cavalry |
| Cavalry | 4 | 1 | Swordsman, Archer |

All units have 100 HP. Strong matchups deal 70 base damage, neutral deal 55.

## Terrain

| Terrain | Defense | Notes |
|---------|---------|-------|
| Plains | — | Move cost 1 for all |
| Mountains | +30% | Cost 3 for infantry/archers, impassable for cavalry |
| Forest | +30% | Cost 2 for all |
| Water | — | Impassable for all |

## Combat

- Damaged units deal less damage (scales down to 50% at low HP)
- Defenders counter-attack if the attacker is within their range
- Melee units adjacent to a defender provide flanking bonuses (+10-25%)
- An ally directly behind the defender scores a backstab (+25%)
- Click **Help** in-game for full damage tables and mechanics details

## AI Opponent

Two difficulty levels:
- **Normal** — Greedy positional AI with focus fire, terrain awareness, flanking, and retreat logic
- **Hard** — Adds coordinated kill targeting, precise danger mapping, frontline support, and archer safety
