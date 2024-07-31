# SavePoint
The same than [Talk](Talk.md), but the [SetText](../../../SetText/SetText.md) input string is hardcoded. The string contains a [prompt](../../../SetText/Individual%20commands/Prompt.md) command where the confirm option leads to a line with a [save](../../../SetText/Individual%20commands/Save.md) command which will actually saves the game.

## args meaning
None.

## SetUp
The [SavePoint](../ObjectTypes/SavePoint.md) object is the only object where an interaction is possible and it is this interaction which is set to its `interacttype`. Check the `SavePoint` object documention to learn more about how it can triggers this interaction.

## Interact
Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being `|`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,4||`[bleep](../../../SetText/Individual%20commands/Bleep.md)`,2,1,1|` + `menutext[4]` (the save prompt text) + `|`[prompt](../../../SetText/Individual%20commands/Prompt.md)`,menu,0.7,2,7,78,5,6|`:

- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- This transform as the parent (or the `shopkeeper` one if `args` is `buy`) and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) called on them with the position being this position and with forceback.

## [GetDialogue](../Notable%20methods/GetDialogue.md)
This method should never be called on an `NPC` with this interaction because doing so will cause the error message to appear with the exception of `overridediag` not being negative, but this never happens under normal gameplay because it relates to an unused feature.
