# Battle strategy list type confirmation handling
This page describes the logic [SetItem](../SetItem.md) has when handling a [listtype](../../../ItemList/listtype.md) of [Battle strategy](../../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md).

The logic depends on the `listvar` option:

## 271 (Swap)
This acts as if 0 (Switch) was selected if there is 2 or less `playerdata`.

Otherwise:

- [itemarea](../../Player%20UI/AttackArea.md) is set to `SingleAlly`
- [currentchoice](../Actions.md) is set to `Swap`
- `helpboxid` is set to -1
- `excludeself` is set to true
- If `currentturn` is 0, `option` is set to 1
- [GotoSelect](../GotoSelect.md) without overridemax is called

## 2 (Do Nothing)
DoNothing is called which does the following:

- If `playerdata[currentturn].didnothing` is false (the player party member didn't already did nothing), several effects can happen depending on at least 1 instance of a specific [medal](../../../Enums%20and%20IDs/Medal.md) being equipped on the `currentturn` player party member (more can stack, these aren't mutually exclusive):
    - `Meditation`: instance.`tp` is incremented by the amount of medals equipped then clamped from 0 to instance.`maxtp`. This is followed by the `Heal2` sound being played and `MagicUp` particles playing at `playerdata[currentturn].battleentity` position + (0.0, 0.5, 0.0)
    - `Prayer`: `playerdata[currentturn].hp` is incremented by the amount of medals equipped * 2 then clamped from 0 to `playerdata[currentturn].maxhp`. This is followed by the `Heal` sound being played (If no `Meditation` is equipped) and `Heal` particles playing at `playerdata[currentturn].battleentity` position + (0.0, 0.5, 0.0)
    - `Reflection`: If no `Prayer` or `Meditation` medals is equipped, the `StatUp` sound is played. This is followed by a [StatEffect](../../Visual%20rendering/StatEffect.md) starting on the battleentity with type 1 (blue, up arrow) and if the player party member doesn't have the [Reflection](../../Actors%20states/BattleCondition/Reflection.md) condition, [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) is called with `Reflection` on the player party member with a turn amount being the amount of medals equipped
- `playerdata[currentturn].didnothing` is set to true which prevents the medals effects to apply on a second do nothing
- [EndPlayerTurn](../../Battle%20flow/EndPlayerTurn.md) is called
- [CancelList](../CancelList.md) is called

## 0 (Switch)
The `Switch` sound is played followed by a [SwitchParty](../../Battle%20flow/Action%20coroutines/SwitchParty.md#switchparty) action coroutine starting switching to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md).

## 1 (Spy)

- [ItemList](../../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this is to workaround a potential [inlist issue](../../../ItemList/inlist%20issue.md))
- `availabletargets` is set to the return of GetTattleable which is all `enemydata` whose `notattle` is false
- If `availabletargets` is empty, PlayBuzzer is called followed by ReloadStrategy being called and invoked another time in 0.1 seconds. The method does the following:
    - [currentaction](../Pick.md) is set to `StrategyList`
    - `helpboxid` is set to -1
    - A couple of [ItemList](../../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../../ItemList/listtype.md): 9 which is the [battle strategy list type](../../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
        - `listammount`: 6
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../../ItemList/ShowItemList.md) is called with 9 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
    - [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- Otherwise:
    - [CreateHelpBox](../../Visual%20rendering/CreateHelpBox.md) with id 0 is called
    - [itemarea](../../Damage%20pipeline/AttackProperty.md) is set to `SingleEnemy`
    - `maxoptions` is set to the length of `availabletargets`
    - [currentaction](../Pick.md) is set to `SelectEnemy`

## 3 (Flee)
If `canflee` is true, a [TryFlee](../../Battle%20flow/Action%20coroutines/TryFlee.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md). This ends the handler.

If it's false however:

- [CancelList](../CancelList.md) is called
- [ItemList](../../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this is to workaround a potential [inlist issue](../../../ItemList/inlist%20issue.md))
- [SetText](../../../SetText/SetText.md) is called in [dialogue mode](../../../SetText/Dialogue%20mode.md) using `|`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,4||`[spd](../../../SetText/Individual%20commands/Spd.md)`,0|` followed by `menutext[73]` (a message informing the player they cannot flee) as the text and the following:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - linebreak of MainManager.`messagebreak`
    - No tridimensional
    - position of Vector3.zero
    - No cameraoffset
    - size of Vector2.one
    - No parent
    - No caller
- A `WaitForMessage` coroutine is started which yields as long as the [message](../../../SetText/Notable%20states.md#message) lock is grabbed followed by calling [UpdateText](../../Visual%20rendering/UpdateText.md) followed by a frame yield
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
