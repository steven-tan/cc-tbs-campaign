# Fantasy Tactical Battle

A turn-based tactical combat game on a hex grid, inspired by Advance Wars. Two players take turns commanding infantry and archers in hotseat mode.

**[Play it here](https://steven-tan.github.io/cc-tbs-initial/)**

## How to Play

- **Select** a unit by clicking it on your turn (Blue goes first)
- **Move** by clicking a highlighted hex
- **Attack** enemies highlighted in red — you can fire before or after moving
- **End Turn** when you're done with all your units
- Eliminate all enemy units to win

## Units

| Unit | Symbol | Range | Move | HP |
|------|--------|-------|------|----|
| Infantry | I | 1 | 3 | 100 |
| Archer | A | 2 | 3 | 100 |

## Terrain

| Terrain | Defense | Move Cost |
|---------|---------|-----------|
| Plains | — | 1 |
| Mountains | +30% | 2 |

## Combat Notes

- Damaged units deal less damage (scales down to 50% at 1 HP)
- Defenders counter-attack, but only if the attacker is within their range
- Mountains reduce incoming damage by 30%
- Click the **Help** button in-game for the full damage table
