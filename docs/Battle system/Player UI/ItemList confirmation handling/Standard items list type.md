# Standard items list type confirmation handling
This page describes the logic [SetItem](../SetItem.md) has when handling a [listtype](../../../ItemList/listtype.md) of [standard items](../../../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md).

- We first check if the item is usable. There are 2 ways in which an item may not be usable:
    - Any `ItemUse` from [itemdata](../../../TextAsset%20Data/Items%20data.md#itemdata) has an `UsageType` of `None`
    - The [AttackArea](../../Player%20UI/AttackArea.md) from [itemdata](../../../TextAsset%20Data/Items%20data.md#itemdata) is `AllEnemies` or `SingleEnemy` and [GetAvaliableTargets](../../Actors%20states/Targetting/GetAvaliableTargets.md) for attackid -1 without onlyground or onlyground, but with excludeunderground causes `availabletargets` to become empty (in other words, the item affects one or all enemy party members, but there's no one above ground to be targetted)
- If the item isn't usable, ReturnToMainSelect is called which does the following:
    - The `Cancel` sound is played on AudioSource 10
    - [currentaction](../Pick.md) is set to `BaseAction`
    - `option` is set to `lastoption`
    - `selecteditem` is set to -1
    - DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
    - [SetMaxOptions](../SetMaxOptions.md) is called
    - [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- Otherwise, if the item is usable:
    - [currentchoice](../Actions.md) is set to `Item`
    - [itemarea](../../Player%20UI/AttackArea.md) is set to the one from [itemdata](../../../TextAsset%20Data/Items%20data.md#itemdata)
    - `helpboxid` is set to -1
    - [GotoSelect](../GotoSelect.md) is called without overridemax
