# [WaspKing](../../Enemy%20actions/Enemies/WaspKing.md) EventDialogue
`WaspKing` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is mostly a mini cutscene supplementing their death, but it also dictates how the battle will end depending on the situation so it ends up switching to a [terminal flow](../Update%20flows/Terminal%20flow.md).

## Logic sequence

- The `enemydata` index of `WaspKing` is obtained
- `WaspKing` animstate set to 116
- If `WaspKing`'s `hologram` is false, FadeMusic is called with 0.03 fadespeed
- Yield for a second
- Yield for 0.5 seconds
- `WaspKing`'s [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to -1
- If `WaspKing`'s `hologram` is false:
    - [ExitBattle](../Terminal%20wrappers/ExitBattle.md) is called
- Otherwise:
    - [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) is called with addexp without skipids
