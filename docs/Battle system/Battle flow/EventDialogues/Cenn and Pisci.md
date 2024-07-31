# [Cenn](../../Enemy%20actions/Enemies/Cenn.md) and [Pisci](../../Enemy%20actions/Enemies/Pisci.md) EventDialogue
This event dialogue occurs either by [Cenn](../../Enemy%20actions/Enemies/Cenn.md) or [Pisci](../../Enemy%20actions/Enemies/Pisci.md)'s action pre move logic when [flags](../../../Flags%20arrays/flags.md) 497 is false (the player haven't beaten them for the first time) or by either of them dying due to their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) which is assumed to be configured to this EventDialogue. Their `eventondeath` gets disabled in [StartBattle](../../StartBattle.md) if `flags` 497 is true meaning this EventDialogue only manages the ending of the first fight.

### EventDialogue 15
This EventDialogue will make sure to thoroughly kill both `Cenn` and `Pisci` through a mini cutscene and call [ExitBattle](../Terminal%20wrappers/ExitBattle.md) to switch the battle to a [terminal flow](../Update%20flows/Terminal%20flow.md):

- The `enemydata` id of both `Cenn` and `Pisci` are obtained
- `Cenn` and `Pisci` each gets [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on them
- Yield for 0.5 seconds
- [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[163]`
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `Cenn`
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[164]`
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `Pisci`
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- `Flee` sound plays
- `Cenn` and `Pisci`'s `flip` gets set to true
- `Cenn` and `Pisci` moves through a [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) to (15.0, 0.0, 0.0) with 50.0 frametime with changeanim
- Both `Cenn` and `Pisci` has their defeated counter in their [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) set to 0 (this cancels what [CheckDead](../Action%20coroutines/CheckDead.md) would have done in case the EventDialogue was triggered through [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath))
- Yield all frames until `Cenn`'s `forcemoving` is done (in theory, `Pisci`'s would be done before)
- [CleanKill](../../Actors%20states/CleanKill.md) called on both `Cenn` and `Pisci` leaving them at (15.0, 0.0, 0.0) as a blank enemy shell
- Yield for a frame
- [ExitBattle](../Terminal%20wrappers/ExitBattle.md) called changing the battle to a [terminal flow](../Update%20flows/Terminal%20flow.md)
