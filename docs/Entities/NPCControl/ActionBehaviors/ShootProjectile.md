# ShootProjectile
Attacks the player by shooting an OverworldProjectile ???

## Frequency meaning
None.

## DoBehavior
If `returntoheight` is true, the entity.`height` is set to a lerp from the existing one to its `initialheight` with a factor of 0.1.

If the entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity without smoothing and without changing the [animstate](../../EntityControl/Animations/animstate.md).

Finally, a ShootProjectile coroutine is started and set to `behaviorroutine`.

## ShootProjectile
This is a private coroutine specific to this behavior. It receives the behavior type and the coroutine call is assigned to `behaviorroutine` which allows the NPCControl to stop it if needed.

- [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 2 (a red ! mark) for 60 frames
- If `projectiles` is null, it is initalised to a new List of OverworldProjectile
- All null elements in `projectiles` are removed
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity to face the player
- From there, the projectile is fired depending on the entity.[animid](../../../Enums%20and%20IDs/AnimIDs.md#animids). See the section below for more details. If this is not an animid with a projectile logic, the following is done:
  - entity.[animstate](../../EntityControl/Animations/animstate.md) is set to entity.`basestate`
  - If we aren't `inrange`, entity.`overrideonlyflip` is set to false
  - A frame is yielded
  - [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
  - The coroutine is exited early with a yield break
- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to entity.`basestate`
- If we aren't `inrange`, entity.`overrideonlyflip` is set to false
- A frame is yielded
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
- If we still aren't `inrange`, [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 1 (? mark) for 60 frames

### Projectile creation
`DeadLanderA`:
TODO

`WaspBomber`:
TODO

`ToeBiter`:
TODO

`SneilEnemy`:
TODO

`BeeBot`:
TODO

`Turret`:
TODO

`WaspScout`:
TODO

`LeafbugArcher`:
TODO

`Bandit`:
TODO

`ChomperBrute`:
TODO

`WildChomper`:
TODO
