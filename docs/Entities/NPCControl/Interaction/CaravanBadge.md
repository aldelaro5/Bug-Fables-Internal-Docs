# CaravanBadge
Interacting with a Caravan prize [medal](../../../Enums%20and%20IDs/Medal.md).

## data meaning
- `data[0]`: An [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) corresponding to the parent of the [SetText](../../../SetText/SetText.md) call.
- `data[1]`: The [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to call [SetText](../../../SetText/SetText.md) with.

## args meaning
None.

## SetUp
If this is an `NPC`, the `pusher`, [GetDialogue](../Notable%20methods/GetDialogue.md) call and [SetInitialBehavior](../Notable%20methods/SetInitialBehavior.md) call do not happen.

[CaravanMedalSet](../Notable%20methods/CaravanMedalSet.md) is then called without reroll.

## HasHiddenItem
Any `Object` with this interaction will always return false, but this never happens under normal gameplay.

## LateUpdate (Every 3 frames, non `dummy` and the entity is `incamera`)
This mainly determines if we should create or destroy the description window. It is created with [CreateDescWindow(true)](CreateDescWindow.md) if all of the following are true:
- [message](../../SetText/Notable%20states.md#message) is released
- We aren't in a `pause`
- We are `inrange`
- The first NPCControl (if it exists) in the interact list of the player is this one
- `descwindow` is null
- The `insideid` matches the current one

On top of this, the `showmoney` is set to 1.0 and 0.0 otherwise (this shows and hides the berry count HUD element accordingly).

If we didn't created the description window, it is destroyed via [DestroyDescWindow](DestroyDescWindow.md) if `descwindow` isn't null, [message](../../SetText/Notable%20states.md#message) is released and at least one of the following is true:
- The `insideid` doesn't match the current one
- We are in a `pause`
- There are no NPCControl in the player interact list or the first one is not this NPCControl
- MainManager's `itempicked` is true

## Interact
First, if the entity `item` is true, the entity `animstate` is set to its `itemstate`.

The [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to the entity `animstate`

If the entity `animid` is 2: 
- [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to the buying price of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to 35.
- [flagstring](../../../Flags%20arrays/flagstring.md) 0 to be the name of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to `menutext[59]` (?????).

Otherwise:
- [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to the buying price of the [item](../../../Enums%20and%20IDs/Items.md) * `mmulti` then ceiled whose id is the entity `animstate` and type being its `animid`.
- [flagstring](../../../Flags%20arrays/flagstring.md) 0 to be the name of the [item](../../../Enums%20and%20IDs/Items.md) whose id is the entity `animstate` and type being its `animid`.

After this, calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being the one whose [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) is `data[1]`:
- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- size is Vector3.one
- The transform of the entity whose [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) is `data[0]` as the parent and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#FaceTowards) called on them with the position being the parent of the SetText call started earlier and with forceback.