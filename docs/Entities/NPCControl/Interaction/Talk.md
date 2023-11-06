# Talk
The interaction used when talking to an NPC ???

## args meaning
None.

## LateUpdate
TODO

## Interact
Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being the current one obtained with [GetDialogue](../GetDialogue.md):
- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- size is Vector3.one
- This transform as the parent and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) called on them with the position being this position and with forceback.