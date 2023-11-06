# SavePoint
Interaction to save the game using a [SavePoint](../ObjectTypes/SavePoint.md) object.

## args meaning
None.

## Interact
Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being `|boxstyle,4||bleep,2,1,1|` + `menutext[4]` (the save prompt text) + `|prompt,menu,0.7,2,7,78,5,6|`:
- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- This transform as the parent (or the `shopkeeper` one if `args` is `buy`) and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) called on them with the position being this position and with forceback.

## [GetDialogue](../Notable%20methods/GetDialogue.md)
This method should never be called on an `NPC` with this interaction because doing so will cause the error message to appear with the exception of `overridediag` not being negative, but this never happens under normal gameplay because it relates to an unused feature.