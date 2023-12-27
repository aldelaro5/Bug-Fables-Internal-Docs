# SetItem
This method is a special handler whenever a confirmation of an [ItemList](../../ItemList/ItemList.md) occurs. The [listtype](../../ItemList/listtype.md) handled here are only the ones related to BattleControl except for the [chompy ribbons list type](../../ItemList/List%20Types%20Group%20Details/Chompy%20Ribbons%20List%20Type.md) as this one is handled by the [Chompy](../Battle%20flow/Action%20coroutines/Chompy.md) action coroutine. The ItemList handled are only the ones that were setup by [GetChoiceInput](GetChoiceInput.md)

The method receives the `listvar` option that was confirmed which is set to `selecteditem`. `turncooldown` is set to 5.0 and `option` to 0.

From there, this is where the list type is handled. Only 3 are possible:

- [Battle strategy list type](../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
- [Skills list type](../../ItemList/List%20Types%20Group%20Details/Skills%20List%20Type.md)
- [Standard items list type](../../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md)

The sections below describes how each is handled. No matter which one, [UpdateText](../Visual%20rendering/UpdateText.md) is called after.

## Battle strategy list type
The logic depends on the `listvar` option:

### 271 (Swap)
This acts as if 0 (Switch) was selected if there is 2 or less `playerdata`.

Otherwise:

- `itemarea` is set to `SingleAlly`
- `currentchoice` is set to `Swap`
- `helpboxid` is set to -1
- `excludeself` is set to true
- If `currentturn` is 0, `option` is set to 1
- GotoSelect without overridemax is called (see the section below for more details)

### 2 (Do Nothing)
DoNothing is called which does the following:

- If `playerdata[currentturn].didnothing` is false (the player party member didn't already did nothing), several effects can happen depending on at least 1 instance of a specific [medal](../../Enums%20and%20IDs/Medal.md) being equipped on the member (more can stack, these aren't mutually exclusive):
    - `Meditation`: instance.`tp` is incremented by the amount of medals equipped then clamped from 0 to instance.`maxtp`. This is followed by the `Heal2` sound being played and `MagicUp` particles playing at `playerdata[currentturn].battleentity` position + (0.0, 0.5, 0.0)
    - `Prayer`: `playerdata[currentturn].hp` is incremented by the amount of medals equipped * 2 then clamped from 0 to `playerdata[currentturn].maxhp`. This is followed by the `Heal` sound being played (If no `Meditation` is equipped) and `Heal` particles playing at `playerdata[currentturn].battleentity` position + (0.0, 0.5, 0.0)
    - `Reflection`: If no `Prayer` or `Meditation` medals is equipped, the `StatUp` sound is played. This is followed by a [StatEffect](../Visual%20rendering/StatEffect.md) starting on the battleentity with type 1 (blue, up arrow) and if the player party member doesn't have the `Reflection` condition, [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with `Reflection` on the player party member with a turn amount being the amount of medals equipped
- `playerdata[currentturn].didnothing` is set to true which prevents the medals effects to apply on a second do nothing
- [EndPlayerTurn](../Battle%20flow/EndPlayerTurn.md) is called
- [CancelList](CancelList.md) is called

### 0 (Switch)
The `Switch` sound is played followed by a [SwitchParty](../Battle%20flow/Action%20coroutines/SwitchParty.md#switchparty) action coroutine starting switching to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow).

### 1 (Spy)

- [ItemList](../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this is to workaround a potential [inlist issue](../../ItemList/inlist%20issue.md))
- `availabletargets` is set to the return of GetTattleable which is all `enemydata` whose `notattle` is false
- If `availabletargets` is empty, PlayBuzzer is called followed by ReloadStrategy being called and invoked another time in 0.1 seconds. The method does the following:
    - `currentaction` is set to `StrategyList`
    - `helpboxid` is set to -1
    - A couple of [ItemList](../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../ItemList/listtype.md): 9 which is the [battle strategy list type](../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
        - `listammount`: 6
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../ItemList/ShowItemList.md) is called with 9 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
    - [UpdateText](../Visual%20rendering/UpdateText.md) is called
- Otherwise:
    - [CreateHelpBox](../Visual%20rendering/CreateHelpBox.md) with id 0 is called
    - `itemarea` is set to `SingleEnemy`
    - `maxoptions` is set to the length of `availabletargets`
    - `currentaction` is set to `SelectEnemy`

### 3 (Flee)
If `canflee` is true, a [TryFlee](../Battle%20flow/Action%20coroutines/TryFlee.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow). This ends the handler.

If it's false however:

- CancelList is called which resets `option` to `lastoption`, `currentaction` to `BaseAction` and `turncooldown` to 5.0. It also calls [SetMaxOptions](SetMaxOptions.md) and [UpdateText](../Visual%20rendering/UpdateText.md#updatetext)
- [ItemList](../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this is to workaround a potential [inlist issue](../../ItemList/inlist%20issue.md))
- [SetText](../../SetText/SetText.md) is called in [dialogue mode](../../SetText/Dialogue%20mode.md) using `|`[boxstyle](../../SetText/Individual%20commands/Boxstyle.md)`,4||`[spd](../../SetText/Individual%20commands/Spd.md)`,0|` followed by `menutext[73]` (`|`[boxstyle](../../SetText/Individual%20commands/Boxstyle.md)`,4||`[halfline](../../SetText/Individual%20commands/Halfline.md)`||`[spd](../../SetText/Individual%20commands/Spd.md)`,0||`[center](../../SetText/Individual%20commands/Center.md)`|Can't escape from this battle!`) as the text and the following:
    - [fonttype](../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - linebreak of MainManager.`messagebreak`
    - No tridimensional
    - position of Vector3.zero
    - No cameraoffset
    - size of Vector2.one
    - No parent
    - No caller
- A `WaitForMessage` coroutine is started which yields as long as the [message](../../SetText/Notable%20states.md#message) lock is grabbed followed by calling [UpdateText](../Visual%20rendering/UpdateText.md) followed by a frame yield
- [UpdateText](../Visual%20rendering/UpdateText.md) is called

## Skills list type

- `maxoptions` is set to the length of `availabletargets` (NOTE: this should be up to date as [SetTargets](../Actors%20states/SetTargets.md) was called the last time [PlayerTurn](../Battle%20flow/PlayerTurn.md) happened while we were in the main vine menu)
- `tempskill` is set to the sent `listvar` option which is the chosen skill id
- `currentchoice` is set to `Skill`
- `helpboxid` is set to the one that comes from the [skilldata](../../TextAsset%20Data/Skills%20data.md#skilldata), but if MainManager.`mashcommandalt` is true (sequential keys is set in the settings):
    - 4 (`TappingKey`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 10 (`RandomTappingBar`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 11 (`RandomPressKeyTimer`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 9 (`RandomPressBar`) is overriden to 12 (`MultiPressBar`)
- `itemarea` is set to the `AttackArea` from `skilldata`
- `excludeself` is set to the the value from `skilldata`
- If `itemarea` is `User`, there is no need to have [GetChoiceInput](GetChoiceInput.md) handle this so it is handled immediately by starting a [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow)
- GotoSelect is called without overridemax (see the section below for details)

## Standard items list type

- We first check if the item is usable. There are 2 ways in which an item may not be usable:
    - Any `ItemUse` from [itemdata](../../TextAsset%20Data/Items%20data.md#itemdata) has an `UsageType` of `None`
    - The `AttackArea` from `itemdata` is `AllEnemies` or `SingleEnemy` and [GetAvaliableTargets](../Actors%20states/GetAvaliableTargets.md) for attackid -1 without onlyground or onlyground, but with excludeunderground causes `availabletargets` to become empty
- If the item isn't usable, ReturnToMainSelect is called which does the following:
    - The `Cancel` sound is played on AudioSource 10
    - `currentaction` is set to `BaseAction`
    - `option` is set to `lastoption`
    - `selecteditem` is set to -1
    - DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
    - [SetMaxOptions](SetMaxOptions.md) is called
    - [UpdateText](../Visual%20rendering/UpdateText.md) is called
- Otherwise, if the item is usable:
    - `currentchoice` is set to `Item`
    - `itemarea` is set to the one from `itemdata`
    - `helpboxid` is set to -1
    - GotoSelect is called without overridemax (see the section below for details)

## GotoSelect
If the confirmation handling needs to move to target selection, a method called GotoSelect is involved receiving a boolean called overridemax which prepares the state for [GetChoiceInput](GetChoiceInput.md) to handle this. Here is the logic of that method depending on the `itemarea` (nothing happens if it's not among them):

### `SingleAlly`

- `maxoptions` is set to the length of `playerdata`
- `currentaction` is set to `SelectPlayer`

### `SingleEnemy`

- If overridemax is false, `maxoptions` is set to the length of `availabletargets`
- `currentaction` is set to `SelectEnemy`

### `AllEnemies`

- `option` is set to 0
- `maxoptions` is set to 1
- `currentaction` is set to `SelectEnemy`

### `AllParty`

- `option` is set to `currentturn`
- `maxoptions` is set to 1
- `currentaction` is set to `SelectPlayer`

### `All`

- `option` is set to 0
- `maxoptions` is set to 1
- `currentaction` is set to `SelectEnemy`
