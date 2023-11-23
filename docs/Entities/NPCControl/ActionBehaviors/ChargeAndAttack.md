# ChargeAndAttack
The same than [ChasePlayer](ChasePlayer.md), but also attacks the player where the actions depends on the [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) when getting closer than a configurable threshold.

## Frequency meaning
The minimum distance to the player at which no attacks will occur. If the distance falls below this number, a ChargeAndAttack coroutine will be started. NOTE: if it is above 10.0 or negative, the value is overriden to 2.0.

## DoBehavior
The logic starts with the same than [ChasePlayer](ChasePlayer.md).

After, if the distance between the NPCControl and the player is less than the frequency (after overriding it as described above), `behaviorroutine` is set to a ChargeAndAttack coroutine that is started.

## ChargeAndAttack
This is a private coroutine specific to this behavior. The call is assigned to `behaviorroutine` which allows the NPCControl to stop it if needed.

- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- entity.`overrideanim` is set to true
- From there, the charge and attack is done depending on entity.[animid](../../../Enums%20and%20IDs/AnimIDs.md#animids). See the section below for more details.
- If we are in a `pause`, `minipause`, `inevent` or we are in a battle and the battle is `inevent`, all frames are yielded until that is no longer the case
- If we aren't `inrange`, [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 1 (? mark) for 60 frames
- entity.`overrideanim` is set to false
- `attacking` is set to false
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called

### Charge and attack logic
`LeafbugClubber`:
TODO

`FlyTrap`:
TODO

`Underling`:
TODO

`Sandworm`:
TODO
