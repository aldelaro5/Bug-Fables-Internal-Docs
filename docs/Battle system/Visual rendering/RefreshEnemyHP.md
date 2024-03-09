# RefreshEnemyHP
This method goes over all the enemy party members and updates the eneablement / displayed text of their battleentity.`hpbar` and `defstat`. This document describes what happens for each `enemydata`.

Nothing happens if the `hpbar` doesn't exist (meaning CreateHPBar had to be called on the battleentity beforehand).

The `hpbar` is always hidden if the enemy has `hidehp` to true while the bar was enabled or if the [message](../../SetText/Notable%20states.md#message) lock is grabbed while `chompyaction` is true.

If it wasn't hidden from the above, the enablement will be updated by testing a series of conditions and it will only actually update if the conditions results in a different enablement than the existing one. Here are all the conditions that must be true for the `hpbar` to remain enabled (it is disabled otherwise):

- `hideenemyhp` is false
- `helpbox` doesn't exist (no description or help text is presented in the bottom left corner)
- Either the enemy was spied according to `librarystuff` or `scopeequipped` is true (The `HPScope` [medal](../../Enums%20and%20IDs/Medal.md) is equipped) or HPBarOnOther returns true (only applicable for a `PitcherFlytrap` [enemy](../../Enums%20and%20IDs/Enemies.md) where it returns whether `Pitcher` was spied or not and for a `KeyR` / `KeyL` [enemy](../../Enums%20and%20IDs/Enemies.md) where it returns whether `EverlastingKing` was spied or not)
- Either `chompyattack` is in progress or all of the following are true:
    - The [message](../../SetText/Notable%20states.md#message) lock is released
    - `action` is false
    - `itemlist` is null (no [Itemlist](../../ItemList/ItemList.md#itemlist) is rendered)
    - The current battle's `inevent` is false
    - `enemy` is false

After updating the enablement, if it's still active, battleentity.`hpbarfont`.`text` is updated to the actualy enemy's `hp`. To update the red bar, the `hpbar`'s first child's first child gets its x scale update to be the enemy's `hp` / `maxhp` (the y/z scale are set to 1.0). To update the yellow bar, the `hpbar`'s first child's second child gets its x scale update to `hpoffset[X]` where X is the length of the `hpbarfont`.`text` - 1 which essentially means if it's one digit, it's 1.1, 1.275 for 2 digits and 1.45 for 3 digits (the y/z scale are set to 1.0).

## TrueDef
The `defstat`'s displayed value is also updated based on the `def`. If the `def` is -1, the displayed value is `?`. NOTE: having the `def` be -1 only visually shows `?`, but it doesn't actually implies the feature of ? defense because this is handled separately by hacing `LimitX10` in the enemy party member's [weakness](../Actors%20states/Enemy%20features.md#weakness).

If it's not -1, it will be the return value of TrueDef which is a method that returns the total defense of the enemy party member. It is is calculated by doing the following:

1. The base defense is determined. It is 0 if the enemy has the [Flipped](../Actors%20states/BattleCondition/Flipped.md) condition and if it doesn't have it, it is the `def` value
2. 1 is added if the enemy has the [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) condition
3. 1 is subtracted if the enemy has the [DefenseDown](../Actors%20states/BattleCondition/DefenseDown.md) condition
4. [defenseonhit](../Actors%20states/Enemy%20features.md#defenseonhit-and-isdefending)'s value is added if the enemy `isdefending` is true
5. 1 is added if the enemy has the [Numb](../Actors%20states/BattleCondition/Numb.md) condition
6. The result is clamped from 0 to 9 and then returned

What's important to note about TrueDef is it is ALWAYS accurate no matter the circumstances because it exhaustively covers all points of defense that can apply to any enemy party member. This is important because [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) may incorrectly ignore some defense sources in various situations.
