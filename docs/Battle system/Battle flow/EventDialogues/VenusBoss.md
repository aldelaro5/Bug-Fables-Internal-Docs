# [VenusBoss](../../Enemy%20actions/Enemies/VenusBoss.md) EventDialogue
`VenusBoss` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is mostly a mini cutscene supplementing their death.

## Logic sequence

- `extraentities[0]` (`Venus`) animstate set to 11 (`Hurt`)
- `VenusBoss` has their `basestate` set to 11 (`Hurt`)
- [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on `VenusBoss`
- If `VenusBoss`'s [position](../../Actors%20states/BattlePosition.md) is `Flying`:
    - Their `droproutine` is set to a new [Drop](../../../Entities/EntityControl/EntityControl%20Methods.md#drop) call
    - Yield all frames until `droproutine` is null (when it completes)
- Camera moves to look near (1.25, 1.5, 2.0)
- Yield for 0.75 seconds
- `cancelupdate` set to true switching to a [terminal flow](../Update%20flows/Terminal%20flow.md) (this is too early to do this, but it is safe because the battle will end here)
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `|`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,1||`[center](../../../SetText/Individual%20commands/Center.md)`||`[shaky](../../../SetText/Individual%20commands/Shaky.md)`||`[halfline](../../../SetText/Individual%20commands/Halfline.md)`||`[size](../../../SetText/Commands.md)`,1.75|` followed by `commondialogue[92]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `extraentities[0]` (`Venus`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- `extraentities[0]` (`Venus`) animstate set to 17 (`WeakBattleIdle`)
- All `enemydata` has their `hp` set to 0 and their `eventondeath` to -1. This will force the death of everyone to happen normally without triggering this EventDialogue again
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called with reset
- Yield for 0.5 seconds
- `checkingdead` set to a new [CheckDead](../Action%20coroutines/CheckDead.md) call which will properly kill every `enemydata`
- Yield until `checkingdead` is null (the CheckDead completed)
