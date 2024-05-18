# `Abombhoney`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Object movement
The object will move to (0.0, 0.0, -0.5) with a ymax of 5.0 instead.

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Done for each player party member whose `hp` is above 0 without an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences) without `plating` and without a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition|null|The player party member|10|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|empty array|false|
|2|Done for each enemy party members without a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition and whose [enemy](../../../Enums%20and%20IDs/Enemies.md) id isn't `Ahoneynation` or `Abomihoney`|null|The enemy party member|10|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|empty array|false|

## Effects on player party members
Only applies to those with an `hp` above 0 without an [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences):

- If the player party member has the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, it is removed via [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) followed by their `shieldenabled` set to false
- If the above didn't apply and they don't have `plating`, DoDamage 1 call happens with them. If they did had a `plating`, the DoDamage call won't happen, but their `plating` is set to false

## Effects on enemy party members

- If the enemy party member has the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, it is removed via [RemoveCondition](../../Actors%20states/Conditions%20methods/RemoveCondition.md) followed by their `shieldenabled` set to false
- Otherwise, if their [enemy](../../../Enums%20and%20IDs/Enemies.md) id is `Ahoneynation` or `Abomihoney`, [Heal](../../Actors%20states/Heal.md) is called on them to heal for 10 `hp`
- Otherwise, DoDamage 2 call happens on them

## Effects on player party

- If not all of the applicable player party members above had a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition:
    - instance.`tp` is increased by instance.`maxtp` * 0.4 floored clamped from 6 to 12. instance.`tp` is then clamped again from 0 to instance.`maxtp`

## Logic sequence

- `Splat1` sound plays
- `Fuse` sound plays
- `Prefabs/Objects/Abombnation` instantiated rooted at the object position
- The object is destroyed
- Yield for 2.0 seconds
- `explosion` particles plays at the `Prefabs/Objects/Abombnation` position
- `Explosion3` sound plays
- `Splat2` sound plays
- `Prefabs/Objects/Abombnation` destroyed
- ShakeScreen with ammount of 0.25 for 0.75 time
- The player party members effects described above are performed
- The enemy party members effects described above are performed
- `Sprites/Objects/splatter` instantiated
- The following is done exactly 3 times (the position and scale of the splatter are hardcoded):
    - A new UI object called `splat` is created childed to the `GUICamera` with the corresponding position and scale that was hardcoded and a sortorder of -5 - index (between 0 and 2) with a flipX of false except for the second one
    - A DialogueAnim is added to `splat` with a `targetscale` of its initial scale and a `shrinkspeed` of 0.15
    - scale of `splat` set to Vector3.zero (so it will grow to its hardcoded scale)
    - `splat`'s `color` set to FFBF00 (bright orange)
    - Yield for 0.025 seconds
- Over the course of 100.0 frames (only on the last 50.0 frames), the alpha of all the `splat` colors goes to 0.0 (transparent)
- All the `splat` are destroyed
- If not all of the applicable player party members above had a [Shield](../../Actors%20states/BattleCondition/Shield.md) condition:
    - [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 2 (TP counter) with ammount of instance.`maxtp` * 0.4 floored clamped from 6 to 12 starting at the attacker + Vector3.up and ending at (0.0, 2.0, 0.0)
    - instance.`tp` is increased by the same ammount then clamped from 0 to instance.`maxtp`
    - `Heal2` sound plays
    - Yield for 0.5 seconds
