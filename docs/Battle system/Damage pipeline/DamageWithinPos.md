# DamageWithinPos
This method is a wrapper around [DoDamage](DoDamage.md) in a player to enemy flow that allows to issue damages to each enemies that matches a filter which includes having their physical (or world [CenterPos](../Actors%20states/CenterPos.md)) be within a certain radius of a point.

The distance check is done with another standalone method called [IsInRadius](IsInRadius.md).

```cs
private void DamageWithinPos(int playerid, int damage, AttackProperty? property, float radius, bool xonly, Vector3 pos)
```

## Parameters

- `playerid`: The `playerdata` index that will be used as the attacker to DoDamage. This must not be negative or an exception will be thrown
- `damage`: The meaning depends on the value and it controls the damageammount and [overrides](DamageOverride.md) parameters to send to DoDamage:
    - 0: DoDamage will be called with a damageammount of the attacker's `atk` and an overrides of {`NoCounter`, `NoDamageAnim`, `NoSound`}
    - 1 or higher: DoDamage will be called with the value as the damageammount with overrides as null
    - -1 or below: DoDamage will be called with a damageammount of the attacker's `atk` + the value with overrides as null
- `property`: The [DamageProperty](AttackProperty.md) to send to DoDamage
- `radius`: The radius to send to IsInRadius
- `xonly`: The xonly to send to IsInRadius
- `pos`: The pos to send to IsInRadius. There is an overload of the method available where this isn't sent and it defaults to the player party member's physical position

## Procedures
Most of the logic on how the DoDamage call is prepared is explained in the parameters section so the only logic remaining is what determines for each enemy party member if DoDamage will be called on it.

Each enemy party member will be checked. In order for DoDamage to be called on them as the target:

- Their [position](../Actors%20states/BattlePosition.md) must not be `OutOfReach`
- Their `hp` must be above 0
- [IsInRadius](IsInRadius.md) returns true with the following parameters:
    - pos: pos
    - entity: The enemy party member
    - radius: radius
    - getair: true
    - xonly: xonly

Any enemy party members that do not fufill those conditions won't have damage dealt to them.
