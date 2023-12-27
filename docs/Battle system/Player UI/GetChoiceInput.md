# GetChoiceInput
This method manages the UI naviguation called from [PlayerTurn](../Battle%20flow/PlayerTurn.md) when `turncooldown` expired.

The logici depends on the `currentaction` and most of the logic involves input handling through the different menus. The sections in this page corresponds to the logic for the [Pick](Pick.md) values handled by this method (the others are handled somewhere else such as MainManager in the case of an [ItemList](../../ItemList/ItemList.md)).

## `BaseAction`
All of the logic present here involves handling inputs. The input isn't processed if `caninputcooldown` hasn't expired yet and nothing happens except decreasing the cooldown by MainManager.`framestep`.

### Input 7 (toggle HUD)
This input is handled specifically where pressing it will set `idletimer` to 250 when pressed which will show the `hexpcounter` without needing to wait 200 frames. Any other input will cause the `idletimer` to be set to 0 which resets the timer.

### Input 2 / 3 (left / right)
These inputs changes the `option` by one depending on the direction (decrement if right, increment if left with wrap around from 0 to `maxoptions` - 1). 

Either input will have `caninputcooldown` set to 2.0 frames, a `Scroll` sound played on `sounds[10]` and [UpdateText](../Visual%20rendering/UpdateText.md) being called.

### Input 5 (cancel)
This input is only processed if [GetFreePlayerAmmount](../Actors%20states/GetFreePlayerAmmount.md) returns more than 1 player being free to act.

- `caninputcooldown` is set to 2.0 frames
- The `Money` sound is player with a pitch of 1.0 + `currentturn` / 10.0 which shifts the pitch slightly using a rate that depends on the selected player that we are switching away from
- `lastoption` is set to `option`
- `currentturn` is set to -1 which unselects the player (the new one will be cycle on the next [player phase](../Battle%20flow/Update.md#player-phase) update)

### Input 6 (switch party)
This input is only processed when AllPartyFree returns true (all `playerdata` have a `cantmove` of 0 or below) or GetAlivePlayerAmmount returns exactly 1 (there is only one player with an `hp` above 0 with no `eatenby`). NOTE: this implies that it is possible to switch after an action if that action killed all, but one party member.

- `caninputcooldown` is set to 2.0
- The `Switch` sound is played
- The [SwitchParty](../Battle%20flow/Action%20coroutines/SwitchParty.md) action coroutine is started (without fast) which changes to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow)

### Input 4 (confirm)

- `caninputcooldown` is set to 2.0
- instance.`inputcooldown` is set to 2.0
- A `Confirm` sound is played on `sounds[10]`
- `lastoption` is set to `option`
- `vineoption` is set to `maxoptions`
- `lastaction` is set to `option`
- `currentchoice` is set to the `Actions` value whose numerical value is `option`

From there, what happens depends on `currentchoice` and after, [UpdateText](../Visual%20rendering/UpdateText.md) is called.

#### `Attack`

- [SetTargets](../Actors%20states/SetTargets.md) is called
- If there's no `availabletargets`, PlayBuzzer is called
- Otherwise:
    - `maxoptions` is set to the length of `availabletargets`
    - `currentaction` is set to `SelectEnemy`
    - `itemarea` is set to `SingleEnemy`
    - `option` is set to 0

#### `Skill`

- `excludeself` is set to false
- `playerdata[currentturn].lockskills` is true or it has the `Taunted` or `Inked` [condition](../Actors%20states/Conditions.md), PlayBuzzer is called
- Otherwise:
    - `currentaction` is set to `SkillList`
    - MainManager.[RefreshSkills](../RefreshSkills.md) is called
    - A couple of [ItemList](../../ItemList/ItemList.md) fields are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../ItemList/listtype.md): -`currentturn` - 1 which is the [skills list type](../../ItemList/List%20Types%20Group%20Details/Skills%20List%20Type.md) of the corresponding player party member
        - `listammount`: 5
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../ItemList/ShowItemList.md) is called with -`currentturn` - 1 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell

#### `Item`

- `excludeself` is set to false
- [GetAvaliableTargets](../Actors%20states/GetAvaliableTargets.md) is called with onlyground without onlyfront using -1 as the acttackid
- If instance.`items[0]` is empty (no standard items) or `playerdata[currentturn].lockitems` is true or it has the `Taunted` or `Sticky` condition, PlayBuzzer is called
- Otherwise:
    - `currentaction` is set to `ItemList`
    - A couple of [ItemList](../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
        - `storeid`: 0
        - [listtype](../../ItemList/listtype.md): 0 which is the [standard items list type](../../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md)
        - `listammount`: 5
        - `listdesc`: true
        - `listsell`: false
    - [ShowItemList](../../ItemList/ShowItemList.md) is called with 0 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
    - [UpdateText](../Visual%20rendering/UpdateText.md) is called

#### `Strategy`

- `currentaction` is set to `StrategyList`
- A couple of [ItemList](../../ItemList/ItemList.md) field are initialised (with an instance.`inputcooldown` of 5.0):
    - `storeid`: 0
    - [listtype](../../ItemList/listtype.md): 9 which is the [battle strategy list type](../../ItemList/List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)
    - `listammount`: 6
    - `listdesc`: true
    - `listsell`: false
- [ShowItemList](../../ItemList/ShowItemList.md) is called with 9 as the list type, MainManager.`defaultlistpos` as the position with showdescription and without sell
- [UpdateText](../Visual%20rendering/UpdateText.md) is called

#### `Relay`

- If there's 1 or less `playerdata` or `playerdata[currentturn].locktri` or `haspassed` is true or it has the `Taunted` [condition](../Actors%20states/Conditions.md), PlayBuzzer is called
- Otherwise:
    - `excludeself` is set to true
    - `option` is set to 0
    - `helpboxid` is set to -1
    - `maxoptions` is set to the length of `playerdata`
    - `itemarea` is set to `SingleAlly`
    - `option` is incremented with wrap around to `maxoptions` - 1 repeatedly via IncreaseOption until its value is `currentturn`
    - `currentaction` is set to `SelectPlayer`

#### `Swap`

- `excludeself` is set to true
- `option` is set to 0
- `helpboxid` is set to -1
- `maxoptions` is set to the length of `playerdata`
- `option` is incremented with wrap around to `maxoptions` - 1 repeatedly via IncreaseOption until its value is `currentturn`
- `currentaction` is set to `SelectPlayer`

## `SelectEnemy`
[CreateHelpBox](../Visual%20rendering/CreateHelpBox.md) is called followed by `excludeself` being set to false.

The rest is input handling logic.

### Input 2 / 3 (left / right)
These 2 inputs are only processed if `itemarea` isn't `AllParty`, `AllEnemies` or `All`.

These inputs changes the `option` by one depending on the direction (decrement if left, increment if right with wrap around from 0 to `maxoptions` - 1 using DecreaseOption and IncreaseOption). 

Additionally, a `Scroll` sound played on `sounds[10]` before changing the `option` and [UpdateText](../Visual%20rendering/UpdateText.md) is called after changing it.

### Input 5 (cancel)
ReturnToMainSelect is called which does the following:

- The `Cancel` sound is played on AudioSource 10
- `currentaction` is set to `BaseAction`
- `option` is set to `lastoption`
- `selecteditem` is set to -1
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- [SetMaxOptions](SetMaxOptions.md) is called
- [UpdateText](../Visual%20rendering/UpdateText.md) is called

### Input 4 (confirm)
A `Confirm` sound is played on `sounds[10]` followed by `target` being set to `option`.

What happens after depends on the `currentchoice` (nothing happen if it's not among these `Actions`).

#### `Attack`
A [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) on `playerdata[currentturn].battleentity` using -1 as the action id.

#### `Item`
CheckItemUse is called with the `selecteditem` which ends starting a [UseItem](../Battle%20flow/Action%20coroutines/UseItem.md) action coroutine changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow).

#### `Skill`

- `lastskill` is set to `selecteditem`
- A [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) on `playerdata[currentturn].battleentity` using `selecteditem` as the action id.

#### `Strategy`

- [ItemList](../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this workarounds a potential [inlist issue](../../ItemList/inlist%20issue.md))
- `currentaction` is set to `BaseAction`
- A [Tattle](../Battle%20flow/Action%20coroutines/Tattle.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) if `disablespy` is false.

## `SelectPlayer`
All the logic present here is input handling logic.

### Input 2 / 3 (left / right)
These 2 inputs are only processed if `itemarea` isn't `AllParty`, `AllEnemies` or `All`.

These inputs changes the `option` by one depending on the direction (decrement if left, increment if right with wrap around from 0 to `maxoptions` - 1 using DecreaseOption and IncreaseOption). However, there are 2 exceptions (only one is applied)

- If `excludeself` is true, the increment/decrement is redone over and over until `option` isn't `currentturn` (meaning the player that will be selected isn't the one performing an action)
- Otherwise, if PartyInOrder returns false (meaning more than 1 member exists and swaps occured such that it is no longer in one of the 3 expected order), the reverse change will be done instead

TODO: something is suspicious here: it feels like depending on things, it could be possible to break player selection combined with swaps...

Additionally, a `Scroll` sound played on `sounds[10]` before changing the `option` and [UpdateText](../Visual%20rendering/UpdateText.md) is called after changing it.

### Input 5 (cancel)
ReturnToMainSelect is called which does the following:

- The `Cancel` sound is played on AudioSource 10
- `currentaction` is set to `BaseAction`
- `option` is set to `lastoption`
- `selecteditem` is set to -1
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- [SetMaxOptions](SetMaxOptions.md) is called
- [UpdateText](../Visual%20rendering/UpdateText.md) is called

### Input 4 (confirm)
If `itemarea` is `AllParty`, `option` is set to `currentturn`.

What happens after depends on the `currentchoice` (nothing happen if it's not among these `Actions`).

#### `Item`
CheckItemUse is called with the `selecteditem` which may end starting a [UseItem](../Battle%20flow/Action%20coroutines/UseItem.md) action coroutine changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow).

However, this item usage may be denied which won't cause UseItem to be called and instead, PlayBuzzer will be called.

The condition for the item usage to be allowed is `playerdata[option].eatenby` must not exist and either `playerdata[option].hp` is above 0 or it's not, but the [item data](../../TextAsset%20Data/Items%20data.md) of `selecteditem` has a  `Revive` or `ReviveAll` `ItemUsage`. It is disallowed otherwise.

#### `Relay`
If CanBeRelayed returns true on `option`, a [Relay](../Battle%20flow/Action%20coroutines/Relay.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) otherwise, PlayBuzzer is called.

For CanBeRelayed to return true, all of the following must be true on `playerdata[option]` (false is returned otherwise):

- Its `hp` is above 0
- Its `lockrelayreceive` is false
- It doesn't have the `Numb` [condition](../Actors%20states/Conditions.md)
- It doesn't have the `Sleep` [condition](../Actors%20states/Conditions.md)
- It doesn't have the `Freeze` [condition](../Actors%20states/Conditions.md)
- It doesn't have the `Taunted` [condition](../Actors%20states/Conditions.md)
- It doesn't have the `Sturdy` [condition](../Actors%20states/Conditions.md)
- It doesn't have the `EventStop` [condition](../Actors%20states/Conditions.md)
- It doesn't have an `eatenby`

#### `Swap`
A [SwitchPos](../Battle%20flow/Action%20coroutines/SwitchPos.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) to swap `currentturn` as the called with `option` as the target.

#### `Skill`
For the skill to be used, if `skilldata` of `tempskill` reports that it can only target alive party members, `playerdata[target].hp` must be above 0. Alternatively, if `skilldata` of `tempskill` reports that it can only target dead party members, `playerdata[target].hp` must be 0 or below. This doesn't apply if neither restrictions applies on the skill, but no matter what, `playerdata[target].eatenby` must not exist.

If the skill can be used, a [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow) on `playerdata[currentturn].battleentity` using `tempskill` as the action id.

Otherwise, PlayBuzzer is called