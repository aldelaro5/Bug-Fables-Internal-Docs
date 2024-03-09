# `BaseAction` ChoiceInput logic
This page details the logic of [GetChoiceInput](../GetChoiceInput.md) when [currentaction](../Pick.md) is `BaseAction`.

All of the logic present here involves handling inputs. The input isn't processed if `caninputcooldown` hasn't expired yet and nothing happens except decreasing the cooldown by MainManager.`framestep`.

## Input 7 (toggle HUD)
This input is handled specifically where pressing it will set `idletimer` to 250 when pressed which will show the `hexpcounter` without needing to wait 200 frames. Any other input will cause the `idletimer` to be set to 0 which resets the timer.

## Input 2 / 3 (left / right)
These inputs changes the `option` by one depending on the direction (decrement if right, increment if left with wrap around from 0 to `maxoptions` - 1). 

Either input will have `caninputcooldown` set to 2.0 frames, a `Scroll` sound played on `sounds[10]` and [UpdateText](../../Visual%20rendering/UpdateText.md) being called.

## Input 5 (cancel)
This input is only processed if [GetFreePlayerAmmount](../../Actors%20states/Player%20party%20members/GetFreePlayerAmmount.md) returns more than 1 player being free to act.

- `caninputcooldown` is set to 2.0 frames
- The `Money` sound is player with a pitch of 1.0 + `currentturn` / 10.0 which shifts the pitch slightly using a rate that depends on the selected player that we are switching away from
- `lastoption` is set to `option`
- `currentturn` is set to -1 which unselects the player (the new one will be cycle on the next [player phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#player-phase) update)

## Input 6 (switch party)
This input is only processed when AllPartyFree returns true (all `playerdata` have a `cantmove` of 0 or below) or GetAlivePlayerAmmount returns exactly 1 (there is only one player with an `hp` above 0 with no `eatenby`). NOTE: this implies that it is possible to switch after an action if that action killed all, but one party member TODO: recheck.

- `caninputcooldown` is set to 2.0
- The `Switch` sound is played
- The [SwitchParty](../../Battle%20flow/Action%20coroutines/SwitchParty.md) action coroutine is started (without fast) which changes to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md)

## Input 4 (confirm)

- `caninputcooldown` is set to 2.0
- instance.`inputcooldown` is set to 2.0
- A `Confirm` sound is played on `sounds[10]`
- `lastoption` is set to `option`
- `vineoption` is set to `maxoptions`
- `lastaction` is set to `option`
- [currentchoice](../Actions.md) is set to the `Actions` value whose numerical value is `option`

From there, what happens depends on [currentchoice](../Actions.md) and after, [UpdateText](../../Visual%20rendering/UpdateText.md) is called.

### `Attack`

- [SetTargets](../../Actors%20states/Targetting/SetTargets.md) is called
- If there's no `availabletargets`, PlayBuzzer is called (the attack usage is denied because no targets is available)
- Otherwise:
    - `maxoptions` is set to the length of `availabletargets`
    - [currentaction](../Pick.md) is set to `SelectEnemy`
    - [itemarea](../../Player%20UI/AttackArea.md) is set to `SingleEnemy`
    - `option` is set to 0

### `Skill`

- `excludeself` is set to false
- `playerdata[currentturn].lockskills` is true or it has the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) or [Inked](../../Actors%20states/BattleCondition/Inked.md) condition, PlayBuzzer is called (the skill usage is denied)
- Otherwise:
    - [currentaction](../Pick.md) is set to `SkillList`
    - MainManager.[RefreshSkills](../../RefreshSkills.md) is called
    - A couple of [ItemList](../../../ItemList/ItemList.md) fields are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../../ItemList/listtype.md): -`currentturn` - 1 which is the [skills list type](../../../ItemList/List%20Types%20Group%20Details/Skills%20List%20Type.md) of the corresponding player party member
        - `listammount`: 5
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../../ItemList/ShowItemList.md) is called with -`currentturn` - 1 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell

### `Item`

- `excludeself` is set to false
- [GetAvaliableTargets](../../Actors%20states/Targetting/GetAvaliableTargets.md) is called with onlyground without onlyfront using -1 as the acttackid
- If instance.`items[0]` is empty (no standard items) or `playerdata[currentturn].lockitems` is true or it has the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) or [Sticky](../../Actors%20states/BattleCondition/Sticky.md) condition, PlayBuzzer is called (the item usage is denied)
- Otherwise:
    - [currentaction](../Pick.md) is set to `ItemList`
    - A couple of [ItemList](../../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../../ItemList/listtype.md): 0 which is the [standard items list type](../../../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md)
        - `listammount`: 5
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../../ItemList/ShowItemList.md) is called with 0 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
    - [UpdateText](../../Visual%20rendering/UpdateText.md) is called

### `Strategy`

- [currentaction](../Pick.md) is set to `StrategyList`
- A couple of [ItemList](../../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
    - `storeid`: 0
    - [listtype](../../../ItemList/listtype.md): 9 which is the [battle strategy list type](../../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
    - `listammount`: 6
    - `listdesc`: true
    - `listsell`: false
- [ShowItemList](../../../ItemList/ShowItemList.md) is called with 9 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

### `Relay`

- If there's 1 or less `playerdata` or `playerdata[currentturn].locktri` (unable to relay) or `haspassed` is true (already relayed on the same main turn) or it has the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition, PlayBuzzer is called (the relay is denied)
- Otherwise:
    - `excludeself` is set to true
    - `option` is set to 0
    - `helpboxid` is set to -1
    - `maxoptions` is set to the length of `playerdata`
    - `itemarea` is set to `SingleAlly`
    - `option` is incremented with wrap around to `maxoptions` - 1 repeatedly via IncreaseOption until its value is `currentturn`
    - [currentaction](../Pick.md) is set to `SelectPlayer`

### `Swap`

- `excludeself` is set to true
- `option` is set to 0
- `helpboxid` is set to -1
- `maxoptions` is set to the length of `playerdata`
- `option` is incremented with wrap around to `maxoptions` - 1 repeatedly via IncreaseOption until its value is `currentturn`
- [currentaction](../Pick.md) is set to `SelectPlayer`
