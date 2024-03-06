# SetCondition
This method is the prefered way to add or amend an existing [condition](../Conditions.md). The vast majority of conditions are inflicted or amended using this method with some narrow exceptions. It belongs to MainManager.

```cs
public static void SetCondition(BattleCondition condition, ref BattleData entity, int turns, int fromplayer)
```

### Parameters

- `condition`: The condition to add/amend
- `entity`: The actor to apply the condition
- `turns`: The amount of actor turns to set/amend the condition
- `fromplayer`: UNUSED (there is an overload without it that sends -1)

A `condition` element of a `BattleData` is composed of 2 integers. The first is the `BattleCondition` value and the second is the amount of actor turns it should be in effect.

### Adding a condition
Adding a condition is a simpler operation as it will be added regardless of the resistance to said condition with only some conditions having special effects. This means that for adding a condition, the caller is responsible for verifying the resistence check.

Here are all the special effects for adding a condition:

|Condition|Additional effect|
|--------:|-----------------|
|Shield|battleentity.`shieldenabled` is set to true. Also, If it's a `Player`, the turn amount of the condition is set to 1|
|Poison|If it's a `Player` with a `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md) equipped, the turn amount of the condition is set to 99999|
|Freeze|If it's not a `Player` while `isdefending` is true, it is set to false|
|Numb|If it's not a `Player` while `isdefending` is true, it is set to false. Also, `isnumb` is set to true|
|Sleep|If it's not a `Player` while `isdefending` is true, it is set to false|
|Any other conditions|None|

After the condition has been added, Abs is done on its actor turn amount forcing the amount to not be negative.

### Amending a condition
Amending a condition requires the requested turns value to be above 0.

If the turns value is negative, there's a different requirement and it is passing a ressitance test for the following statuses (it always pass for any other):

- `Freeze`: uses `freezeres`
- `Numb`: uses `numbres`
- `Poison`: uses `poisonres`
- `Sleep`: uses `sleepres`

The test is performed by generating a number between 0.0 and 100.0 inclusive. For the test to pass, the number bust be strictly above the corresponding resistance. If the test fails, the condition won't be amended.

#### Problems with the resistance test
However, this negative turns resistance test is in practice UNUSED and broken. The UNUSED part is because the only time SetCondition is called with a negative turns is when processing certain [item](../../../Enums%20and%20IDs/Items.md)'s [effects](../../../TextAsset%20Data/Items%20data.md):

- `ShavedIce` (its `AddFreeze`)
- `HoneyIceCream` (its `AddFreeze`)
- `IceCream` (its `AddFreeze`)
- `FrozenSalad` (its `AddFreeze`)
- `FrostPie` (its `AddFreeze`)
- `DrowsyCake` (its `AddPoison`)

But DoItemEffect won't actually allow the SetCondition call for these effects if any of the appropriate resistance [medals](../../../Enums%20and%20IDs/Medal.md) are equipped and it turns out this is the only way in practice to increase the resistance fields of a player party member. What this means is in any cases, even if the turns count was negative, the test always passes. This only matters if a negative turns value is passed to SetCondition in other cases than DoItemEffect outside of normal gameplay.

But if it does, there's several problems where the logic is incorrect. Most of the issues is this resistance test differs greatly from [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) because ONLY the resistance check is performed while the damage pipeline performs several other checks.

It is speculated that this feature was left unfinished as an older attempt of implementing user infliction and it can be considered broken in its current state. While the game can run into it with some specific item use, in practice, it doesn't do anything of note by coincidence. Still, in essence, negative turns amount should be considered invalid.

#### When turns is above 0 or the negative turns test pass
If the test pass, the condition will be amended, but how this occurs depends on the condition itself. Some may allow stacking the actor turns, some conditionally and some never. Also, some conditions implicates special side effects.

The following table summarises the logic for all conditions (stacking means increasing the existing actor turn amount by the requested one unless otherwise specified and not stacking means to set the amount to the requested one unless otherwise specified):

|Condition|Stacked for a `Player`?|Stacked for a non `Player`?|Other effects|
|--------:|----------------------|---------------------------|------------|
|AttackUp|Yes|Yes|None|
|DefenseUp|Yes|Yes|None|
|AttackDown|Yes|Yes|None|
|DefenseDown|Yes|Yes|None|
|GradualHP|Yes|Yes|None|
|GradualTP|Yes|Yes|None|
|Fire|Yes (by half the requested amount of actor turns ceiled)|Yes (by half the requested amount of actor turns ceiled)|None|
|poison|Yes if it has the `EternalPoison` [medal](../../../Enums%20and%20IDs/Medal.md) equipped which makes it set to 99999 regardless of the requested amount of actor turns|Yes|None|
|Freeze|Only if `currentchoice` is `Item` (meaning the player party used an item on itself), the condition isn't amended if it already had the requested amount of actor turns or more|No|For a non `Player`, `isdefending` is set to false|
|Numb|Only if `currentchoice` is `Item` (meaning the player party used an item on itself), the condition isn't amended if it already had the requested amount of actor turns or more|No|For a non `Player`, `isdefending` is set to false. Also, `isnumb` is set to true|
|Sleep|Only if `currentchoice` is `Item` (meaning the player party used an item on itself), the condition isn't amended if it already had the requested amount of actor turns or more|No|For a non `Player`, `isdefending` is set to false|
|Shield|No, the turn amount is always set to 1 regardless of the requested one|No|battleentity.`shieldenabled` is set to true|
|Any other conditions|No|No|None|

After the condition has been amended, its turn amount is clamped from 0 to 999999.
