# [EverlastingKing](../../Enemy%20actions/Enemies/EverlastingKing.md#everlastingking) EventDialogue
`EverlastingKing` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is mostly a mini cutscene supplementing their death.

## Logic sequence

- The `enemydata` index of `EverlastingKing` is obtained
- `EverlastingKing`'s [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) is set to -1 to disable it and prevent [CheckDead](../Action%20coroutines/CheckDead.md) from triggering it again
- If `EverlastingKing`'s `droproutine` is in progress and `height` is above 0.15, `droproutine` is set to a new [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) call on them
- `startdrop` is set to true which forces the Drop call from proceeding with the enemy drop immediately
- Yield all frames until `EverlastingKing`'s `droproutine` is done
- `EverlastingKing` animstate set to 18 (`KO`)
- Yield for a second
- Yield for 0.5 seconds
- `EverlastingKing` animstate set to 110
- Yield for a frame
- If `hologram` is false:
    - [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[177]`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: `EverlastingKing`
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) called with addexp and without skipids which will end the battle and switch to a [terminal flow](../Update%20flows/Terminal%20flow.md)
