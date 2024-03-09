# `SelectPlayer` ChoiceInput logic
This page details the logic of [GetChoiceInput](../GetChoiceInput.md) when `currentaction` is `SelectPlayer`.

All the logic present here is input handling logic.

## Input 2 / 3 (left / right)
These 2 inputs are only processed if `itemarea` isn't `AllParty`, `AllEnemies` or `All`.

These inputs changes the `option` by one depending on the direction (decrement if left, increment if right with wrap around from 0 to `maxoptions` - 1 using DecreaseOption and IncreaseOption). However, there are 2 exceptions (only one is applied)

- If `excludeself` is true, the increment/decrement is redone over and over until `option` isn't `currentturn` (meaning the player that will be selected isn't the one performing an action)
- Otherwise, if PartyInOrder returns false (meaning more than 1 member exists and swaps occured such that it is no longer in one of the 3 expected order), the reverse change will be done instead. This works because all combinations where the party is out of order happens to cycle in reverse order (intuitively, think of fixing the first number, your only choices are +1 and -1 in the cycle which locks in the last one and the latter choice is out of order)

Additionally, a `Scroll` sound played on `sounds[10]` before changing the `option` and [UpdateText](../../Visual%20rendering/UpdateText.md) is called after changing it.

## Input 5 (cancel)
ReturnToMainSelect is called which does the following:

- The `Cancel` sound is played on AudioSource 10
- `currentaction` is set to `BaseAction`
- `option` is set to `lastoption`
- `selecteditem` is set to -1
- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- [SetMaxOptions](../SetMaxOptions.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

## Input 4 (confirm)
If `itemarea` is `AllParty`, `option` is set to `currentturn`.

What happens after depends on the `currentchoice` (nothing happen if it's not among these `Actions`).

### `Item`
CheckItemUse is called with the `selecteditem` which may end starting a [UseItem](../../Battle%20flow/Action%20coroutines/UseItem.md) action coroutine changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md).

However, this item usage may be denied which won't cause UseItem to be called and instead, PlayBuzzer will be called.

The condition for the item usage to be allowed is `playerdata[option].eatenby` must not exist and either `playerdata[option].hp` is above 0 or it's not, but the [item data](../../../TextAsset%20Data/Items%20data.md) of `selecteditem` has a  `Revive` or `ReviveAll` `ItemUsage`. It is disallowed otherwise.

### `Relay`
If CanBeRelayed returns true on `option`, a [Relay](../../Battle%20flow/Action%20coroutines/Relay.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md) otherwise, PlayBuzzer is called.

For CanBeRelayed to return true, all of the following must be true on `playerdata[option]` (false is returned otherwise):

- Its `hp` is above 0
- Its `lockrelayreceive` is false
- It doesn't have the `Numb` [condition](../../Actors%20states/Conditions.md)
- It doesn't have the `Sleep` [condition](../../Actors%20states/Conditions.md)
- It doesn't have the `Freeze` [condition](../../Actors%20states/Conditions.md)
- It doesn't have the `Taunted` [condition](../../Actors%20states/Conditions.md)
- It doesn't have the `Sturdy` [condition](../../Actors%20states/Conditions.md)
- It doesn't have the `EventStop` [condition](../../Actors%20states/Conditions.md)
- It doesn't have an `eatenby`

### `Swap`
A [SwitchPos](../../Battle%20flow/Action%20coroutines/SwitchPos.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md) to swap `currentturn` as the called with `option` as the target.

### `Skill`
For the [skill](../../../Enums%20and%20IDs/Skills.md) to be used, if `skilldata` of `tempskill` reports that it can only target alive party members, `playerdata[target].hp` must be above 0. Alternatively, if `skilldata` of `tempskill` reports that it can only target dead party members, `playerdata[target].hp` must be 0 or below. This doesn't apply if neither restrictions applies on the skill, but no matter what, `playerdata[target].eatenby` must not exist.

If the skill can be used, a [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md) action coroutine is started changing to an [uncontrolled flow](../../Battle%20flow/Update%20flows/Uncontrolled%20flow.md) on `playerdata[currentturn].battleentity` using `tempskill` as the action id.

Otherwise, PlayBuzzer is called