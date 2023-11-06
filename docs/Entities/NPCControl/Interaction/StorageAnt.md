# StorageAnt
The interaction when using the Ant Storage Service.

## args meaning
None.

## LateUpdate
If the [message](../../SetText/Notable%20states.md#message) lock is released, we aren't `inevent` and it's not an entity.`item`, the [animstate](../EntityControl/Animations/animstate.md) is set to the `basestate`

## Interact
First, the [flag](../../../Flags%20arrays/flags.md) slot 349 is set to true (using the storage service, allows multiselect).

The text to call [SetText](../../../SetText/SetText.md) with is determined to be `commondialogue[1]` (the storage service text) unless [flag](../../../Flags%20arrays/flags.md) slot 180 is false in which case, `|anim,caller,Happy|` + `commondialogue[97]` is used instead (the tutorial of the storage service).

[flag](../../../Flags%20arrays/flags.md) slot 180 is then set to true.

Finally, calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being the one determined above:
- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- This transform as the parent and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) called on them with the position being this position and with forceback.