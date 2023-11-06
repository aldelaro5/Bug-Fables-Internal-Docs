# Shop
Interacting with a shop related entity (not to be confused with the [ShopKeeper](ShopKeeper.md))

## args meaning
None.

## SetUp
- If this is an `NPC`, the `pusher`, [GetDialogue](../Notable%20methods/GetDialogue.md) call and [SetInitialBehavior](../Notable%20methods/SetInitialBehavior.md) call does not happen.
- If `shopkeeper` isn't null, `mmulti` is set to 1.0 unless `shopkeeper.dialogues[2].y` is above 0.1 in which case, `mmulti` is set to `shopkeeper.dialogues[2].y` / 10.0 instead. TODO: ???
- [CaravanMedalSet](../Notable%20methods/CaravanMedalSet.md) is called without reroll.

## HasHiddenItem
Any `Object` with this interaction will always return false, but this never happens under normal gameplay.

## LateUpdate (Every 3 frames, non `dummy` and the entity is `incamera`)
This logic mainly determines if we should create or destroy the description window. It is created with [CreateDescWindow(true)](CreateDescWindow.md) if all of the following are true:
- [message](../../SetText/Notable%20states.md#message) is released
- We aren't in a `pause`
- We are `inrange`
- The first NPCControl (if it exists) in the interact list of the player is this one
- `descwindow` is null
- The `insideid` matches the current one

On top of this, if the `shopkeeper`'s `dialogues[1].y` isn't -1 TODO ???, the `showmoney` is set to 1.0 and 0.0 otherwise (this shows and hides the berry count HUD element accordingly).

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

From there, the rest acts as if we interacted with the same logic than [ShopKeeper](ShopKeeper.md) with the `args` being `buy`.

## [GetDialogue](../Notable%20methods/GetDialogue.md)
This method should never be called on an `NPC` with this interaction because doing so will cause the error message to appear no matter the circumstances.
