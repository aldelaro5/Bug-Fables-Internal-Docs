# `NumbDart`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Item logic
The folowing [DoDamage](../../Damage%20pipeline/DoDamage.md) call happens:

|attacker|target|damageammount|property|overrides|block|
|---|---|---|---|---|---|
|null|`availabletargets[target]`|2|[Sleep](../../Damage%20pipeline/AttackProperty.md)|empry array|false|

## Object movement
The starting angle of the object in z is 135.0.

As for the movement, it will occur over the course of 25.0 frames instead of 40.0 without rotation and the movement is done via a simple lerp.