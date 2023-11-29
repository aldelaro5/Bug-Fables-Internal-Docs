# Shop
Interacting with a shop related entity.

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

## EntityControl.[Unfix](../../EntityControl/EntityControl%20Methods.md#unfix)
Any NPCControl with this interaction will block the unifixing unless force is true.

## [SetText](../../../SetText/SetText.md)
This interactions affects SetText in several ways:
- [ShopLine](../../../SetText/Individual%20commands/Shopline.md) commands are only processed by [OrganizeLines](../../../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md#organiselines) if the first NPCControl in the player.`npc` list has this interaction
- In [dialogue setup](../../../SetText/Life%20Cycle.md#dialogue-setup), if the caller has this interaction while its `shopkeeper.dialogues[1].y` isn't 1, the berry count HUD will be shown at the start of the call
- In [dialogue setup](../../../SetText/Life%20Cycle.md#dialogue-setup), if the caller has this interaction while we aren't `inevent`, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) will be called on player.entity to face towards the caller.`shopkeeper` instead of the caller
- [Destroydescbox](../../../SetText/Individual%20commands/Destroydescbox.md) won't call DestroyDescWindow if the caller has this interaction unless the command is used in syntax (2)
- [GiveItem](../../../SetText/Individual%20commands/Giveitem.md) will not call CreateDescWindow if the caller has this interaction
- [Kill](../../../SetText/Individual%20commands/Kill.md) will cause SetBadgeShop with refresh to be called on the `shopkeeper` of the entity to kill if the caller has this interaction and the entity to kill's `animid` is 2 (meaning it's a [medal](../../../Enums%20and%20IDs/Medal.md) as it's assumed that this is an `item` entity)

## PlayerControl.LateUpdate
entity.`emoticonid` gets set to 1 (a blue ?) and entity.`emoticoncooldown` gets set to 2.0 if all of the following conditions are met:
- We aren't in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging`, `flying`, `startdig` or in `shield`
- This is the closest NPC in the `npc` list as of the last 3 frames

Additionally, if all of the above are met, the berry count HUD is shown if `shopkeeper.dialogues[1].y` isn't 1 (it's hidden if it is).