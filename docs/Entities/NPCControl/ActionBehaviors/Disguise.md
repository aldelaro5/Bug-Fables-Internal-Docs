# Disguise
Disguised enemies ???

## Frequency meaning
None ???

## Start / SetUp
The disguise is setup which starts by setting the entity's `alwaysactive` to true. Then, the `disguiseobj` is initialised to the transform of `prefabs/Disguises/x` prefab where x is the entity.[AnimID](../../Enums%20and%20IDs/AnimIDs.md). It gets childed to the entity's `sprite` which also gets disabled making only the disguise visible instead of the sprite. The local position is ensured to be Vector3.zero, but the angles and the scales depends on the AnimID (anything not mentioned is left default):
- `Cactus`: scale of (70.0, 25.0, 70.0)
- `Mushroom`: scale of (20.0, 15.0, 25.0)
- `CursedSkull`: scale of 0.5 uniform, local position of (0.0, 0.0, -0.5) and angles of (-90.0, 0.0, 0.0)
- `Plumpling`: scale of 0.2 uniform and angles of (-90.0, 0.0, 180.0)

The `disguiseobj` is then enabled with an expired cooldown. It also causes the entity's `sprite` to be disabled and the `height` to be 0.0 which hides the actual sprite of the entity with no visual offset and it leaves the disguise object visible.

## DoBehavior
First, a special case is handled where this behavior no longer exists, but the `disguiseobj` still does. The only way for this case to happen is if a destroy of the `disguiseobj` was requested by [RespawnEnemy](../Notable%20methods/RespawnEnemy.md) because not only it destroys it, but it also changes the default behavior to [Wander](Wander.md). If this was the previous one and it wasn't the `inrange` one, then it no longer exists after the respawn which makes it fall into this special case. The handling goes as follow:
- TempSpin is called on the entity which sets its `spin` to (0.0, 20.0, 0.0) and then zerored out in 0.2 seconds
- The `disguisecooldown` is set to 120.0
- The entity `overridehright` is set to false
- The `disguiseobj` is disabled

The main logic after handling this depends on the `disguisecooldown` which is an int countdown of the number of update cycles left before the `disguiseobj` is enabled and the entity `sprite` disabled. This occurs if by the end of this cycle, it reaches 0 or below and the opposite happens if it not. If it already reached 0 by the start of this cycle however, the following is done before (and this cycle ends after):
- It will be set to -1
- TempSpin will be called on the entity which sets its `spin` to (0.0, 20.0, 0.0) and then zerored out in 0.2 seconds
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) will be called on the entity.

If the `disguisecooldown` hasn't expired yet, we continue. If it's 20 or above and the distance between the entity `startpos` and this NPCControl is more than 2.0, it's limited to call [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) on the entity, decrement the cooldown and to toggle the entity `flip` if the cooldown is exactly 40 or 80 TODO: huh ???. The cycle then ends like described above.

If the `disguisecooldown` is also less than 20, this is where the main logic is performed before the cycle ends like described above on the entity:
- Call [DetectDirection](../../EntityControl/EntityControl%20Methods.md#DetectDirection) with the `startpos`
- Align the `moverotater` to the `startpos` via a LookAt call
- If a [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) call returns true, [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called to move the entity to its `startpos` ignoring the y component at normal speed with the state being `Walk` and the stopstate being `Idle`. Otherwise, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called setting the [animstate](../../EntityControl/Animations/animstate.md) to its existing one at the end
- The [animstate](../../EntityControl/Animations/animstate.md) is set to the `walkstate`
- The `oldstate` is set to -1
- If the entity `hitwall`, the position is set to the `startpos`, the `rigid` velocity is zeroed out and DeathSmoke particles are played at the position if the entity is `incamera`

## Update (Inactive)
If the `disguisecooldown` expired (it's -1), the `disguiseobj` gets activated with the entity getting some adjustements:
- The `sprite` gets disabled if `campos.z` is above 5.0 (meaning it's too far forward from the camera), it remains enabled otherwise
- `overrideminheight` is set to true
- The `height` is set to 0.0

## LateUpdate (Not a `dummy` and the entity is `incamera`)
If the `freezecooldown` and `dizzytime` expired while we aren't `minipause`, the `height` is set to a lerp from the existing one to `initialheight` or 0.0 with a factor of 1/10 of the game's frametime. What determines the destination height is if the `disguisecooldown` is 0 or above where it's `initialheight`, 0.0 otherwise.

Otherwise, if we are in a `minipause`, then the `disguiseobj` is enabled if `disguisecooldown` is -1, disabled otherwise while the opposite is done on the entity `sprite`. The enablement values only changes when needed instead of every cycle by checking if the `disguiseobj` is enabled or not in regards to `disguisecooldown`.

## Update ([Dizzy](../Notable%20methods/Dizzy.md) `Enemy` override)
The `disguiseobj` is disabled with the `sprite` being enabled.

## [RespawnEnemy](../Notable%20methods/RespawnEnemy.md) (Only if it's the default behavior)
- The `disguiseobj` is destroyed if it was present
- The entity's `speed` is set to 2.0
- The `disguisecooldown` is set to -1
- The default behavior is set to [Wander](Wander.md)
- The entity's `onground` is set to true in 0.5 seconds

This will get further handled by DoBehavior, see the section above for details

TODO: disguise seems complex, needs more clarity on what each parts of the cycle does