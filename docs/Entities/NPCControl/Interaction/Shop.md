# Shop
At loading time, this interaction is only for indicative purposes to build a [shop system](../Shop%20system.md). At runtime after the shop system has been built, this is the interaction of a shelved item which is a `SemiNPC` [item entity](../../EntityControl/Item%20entity.md) and the interaction calls [SetText](../../../SetText/SetText.md) with the shop keeper's buying dialogue.

> NOTE: The [Shop system](../Shop%20system.md) is very complex. Check the shop system documentation to learn more about how this interaction is involved and how a shop structure works.

## args meaning
None.

## Data arrays
See the [shop system](../Shop%20system.md) documentation.

## SetUp
- If this is an `NPC`, the `pusher`, [GetDialogue](../Notable%20methods/GetDialogue.md) call and [SetInitialBehavior](../Notable%20methods/SetInitialBehavior.md) call does not happen.

The rest of the logic is the standard initialisation procedure described in the [shop system](../Shop%20system.md) documentation.

## HasHiddenItem
Any `Object` with this interaction will always return false, but this never happens under normal gameplay.

## Interact
If this is an `item` entity, the entity `animstate` is set to its `itemstate`.

The [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to the entity `animstate`

If the entity `animid` is 2 (it's a medal):

- [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to the buying price of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to 35.
- [flagstring](../../../Flags%20arrays/flagstring.md) 0 to be the name of the [medal](../../../Enums%20and%20IDs/Medal.md) whose id is the entity `animstate` unless [flag](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) which forces it to `menutext[59]` (?????).

Otherwise:

- [flagvar](../../../Flags%20arrays/flagvar.md) 1 is set to the buying price of the [item](../../../Enums%20and%20IDs/Items.md) * `mmulti` then ceiled whose id is the entity `animstate` and type being its `animid`. `mmulti` was set during SetUp using a value from the shop entity data.
- [flagstring](../../../Flags%20arrays/flagstring.md) 0 to be the name of the [item](../../../Enums%20and%20IDs/Items.md) whose id is the entity `animstate` and type being its `animid`.

From there, the rest acts as if we interacted with the same logic than [ShopKeeper](ShopKeeper.md) with the `args` being `buy` which will call [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) using `shopkeeper.dialogues[6].y` as the [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) for the input string (the buying SetText line) with the following:

- [fonttype](../../../SetText/Notable%20states.md#fonttype) of `BubblegumSans`
- `messagebreak` as the linebreak
- No tridimensional
- No position offset
- No camera offset
- size of Vector3.one
- The `shopkeeper` as the parent
- This NPCControl as the caller

Finally, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) called on them to face the `shopkeeper` with force back.

## LateUpdate (Every 3 frames, non `dummy` and the entity is `incamera`)
This logic mainly determines if we should create or destroy the `descwindow`. It is created with [CreateDescWindow(true)](../Notable%20methods/CreateDescWindow.md) if all of the following are true:

- [message](../../../SetText/Notable%20states.md#message) is released
- We aren't in a `pause`
- We are `inrange`
- The first NPCControl (if it exists) in the interact list of the player is this one
- `descwindow` is null
- The `insideid` matches the current one

On top of this before the creation, if the `shopkeeper`'s `dialogues[1].y` is 1 (it's a medal shop), the `showmoney` is set to 1.0 and 0.0 otherwise (this shows and hides the berry count HUD element accordingly).

If we didn't created the description window, it is destroyed via DestroyDescWindow if `descwindow` isn't null, [message](../../../SetText/Notable%20states.md#message) is released and at least one of the following is true:

- The `insideid` doesn't match the current one
- We are in a `pause`
- There are no NPCControl in the player interact list or the first one is not this NPCControl
- instance.`itempicked` is true

### DestroyDescWindow
This is a public method that will undo what [CreateDescWindow](../Notable%20methods/CreateDescWindow.md) did. It is heavily involved in this interaction, but [destroydescwindow](../../../SetText/Individual%20commands/Destroydescbox.md) and [additemtoss](../../../SetText/Individual%20commands/Additemtoss.md) SetText commands may call it (check their documentations to learn more). It only does anything if the `descwindow` exists.

- `descwindow`.shrink is set to true which will progressively shrink the window until it disappears
- The position of the `descwindow` is incremented by a 1/10 of the normalised direction of the camera
- The `descwindow` is destroyed in half a second
- `descwindow` is set to null which ensures the game sees it as not existing
- If we were `inrange` with this interaction, `inrange` is set to false and the player.`npc` is reset to a new list

## [GetDialogue](../Notable%20methods/GetDialogue.md)
This method should never be called on an NPCControl with this interaction because doing so will cause the error message to appear no matter the circumstances.

## EntityControl.[Unfix](../../EntityControl/EntityControl%20Methods.md#unfix)
Any NPCControl with this interaction will block the unifixing unless force is true.

## [SetText](../../../SetText/SetText.md)
This interactions affects SetText in several ways:

- [shopline](../../../SetText/Individual%20commands/Shopline.md) commands are only processed by [OrganizeLines](../../../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md#organiselines) if the first NPCControl in the player.`npc` list has this interaction
- In [dialogue setup](../../../SetText/Life%20Cycle.md#dialogue-setup), if the caller has this interaction while its `shopkeeper.dialogues[1].y` isn't 1 (the shop accepts regular berries as currency), the berry count HUD will be shown at the start of the call
- In [dialogue setup](../../../SetText/Life%20Cycle.md#dialogue-setup), if the caller has this interaction while we aren't `inevent`, [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) will be called on player.entity to face towards the caller.`shopkeeper` instead of the caller itself
- [destroydescbox](../../../SetText/Individual%20commands/Destroydescbox.md) won't call DestroyDescWindow if the caller has this interaction unless the command is used in syntax (2)
- [giveitem](../../../SetText/Individual%20commands/Giveitem.md) will not call CreateDescWindow if the caller has this interaction
- [kill](../../../SetText/Individual%20commands/Kill.md) will cause SetBadgeShop with refresh to be called on the `shopkeeper` of the entity to kill if the caller has this interaction and the entity to kill's `animid` is 2 (meaning it's a [medal](../../../Enums%20and%20IDs/Medal.md) as it's assumed that this is an `item` entity)

## PlayerControl.LateUpdate
entity.`emoticonid` gets set to 1 (a blue ?) and entity.`emoticoncooldown` gets set to 2.0 if all of the following conditions are met:

- We aren't in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging`, `flying`, `startdig` or in `shield`
- This is the closest NPC in the `npc` list as of the last 3 frames

Additionally, if all of the above are met, the berry count HUD is shown if `shopkeeper.dialogues[1].y` isn't 1 (meaning this shop uses regular berry currency). It's hidden if it is 1 (meaning the shop uses Crystal Berries instead of berries).