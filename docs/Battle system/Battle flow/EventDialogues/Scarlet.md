# [Scarlet](../../Enemy%20actions/Enemies/Scarlet.md)
`Scarlet` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is mostly a mini cutscene supplementing their death.

## Logic sequence

- `Scarlet`'s `isnumb` and `isasleep` set to false
- [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on `Scarlet`
- `Scarlet`'s `jumpheight` set to 0.0
- If `Scarlet` has the [Freeze](../../Actors%20states/BattleCondition/Freeze.md) condition, [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on `Scarlet` again
- `Scarlet` animstate set to 11 (`Hurt`)
- `Umbrella` sound plays
- ShakeSrprite called with intensity (0.1, 0.0, 0.0) and 20.0 frametimer
- Yield for 0.25 seconds
- `Scarlet` animstate set to 18 (`KO`)
- Yield for 0.25 seconds
- A new sprite object is created rooted at this enemy position + (0.0, 2.0, -0.1) using the `Sprites/Entities/scarlet[18]` sprite (the umbrella)
- Over the course of 60.0 frames:
    - The umbrella moves to (12.0, 1.0, 0.0) via a BeizierCurve3 with a ymax of 4.0
    - `Scarlet` moves +0.5 in x via a BeizierCurve3 with a ymax of 0.25 with a factor of the current frame / 180.0 clamped from 0.0 to 1.0
    - Before each frame yield, the umbrella Rorate by 20x the game's frametime in z
- The umbrella gets destroyed
- Yield for 0.75 seconds
- [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) cllaed with addexp without any skipids
