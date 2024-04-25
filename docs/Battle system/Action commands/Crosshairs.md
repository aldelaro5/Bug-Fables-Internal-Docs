# Crosshairs


## `data` array

- `data[0]`: ???
- `data[1]`: ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4
- The `Crosshair` sound is played on `sounds[9]` with 0.5 volume on loop
- `commandsprites` is set to a new array of 2 elements where each is the SpriteRenderer of a new UI object named `crossX` where `X` is the index childed to the `battlemap` with a scale of Vector3.one using the `guisprites[41]` sprite with a sortorder of 10 + index on layer 15 (`3DUI`). The local position depends on `data[1]`:
    - If it's at least 0, it's the `enemydata`'s battleentity position at index `data[1]` (truncated to int) + (Vector3.up * (the enemy party member's battleentity.`height` - 1.0) + the enemy party member's `cursoroffset`). However, if the enemy party member's [position](../Actors%20states/BattlePosition.md) is `Underground`, it's the enemy party member's battleentity position + (0.0, 0.5, 0.0)
    - Otherwise (meaning `data[1]` is negative), the logic is the same, but the `enemydata` index is the absolute value of `data[1]` - 1 and the local position doesn't change if the enemy party member's [position](../Actors%20states/BattlePosition.md) is `Underground`
- `commandsprites[0]`.sprite is set to `guisprites[93]`

## [DoCommand](../DoCommand.md) Execution phase

