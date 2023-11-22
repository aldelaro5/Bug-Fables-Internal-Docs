# Disguise
A behavior where the map entity has a special `disguiseobj` that will replace its normal entity.`sprite` appearence until the behavior is switched away from which reveals the entity.`sprite` (usually by going into the `inrange` one). After coming back to this behavior (usually by going out of `inrange`), the entity will move back to its `startpos` and put its disguise on again.

## Frequency meaning
None.

## A general explanation of how disguising works
This behaviors hinges on a special object called the `disguiseobj` which is childed to entity.`sprite` on SetUp.

Due to this structure, it's positioned similarly to how entity.`model` would, but this behavior is made to not involve a [model entity](../../EntityControl/Notable%20methods/AddModel.md) and instead flip flops the enablement of the `disguiseobj` and the entity.`sprite` such that only one of the 2 is enabled at a given time.

However, EntityControl.[Update](../../EntityControl/Update%20process/Unity%20events/Update.md) will think this structure is like a model entity and it will try to sync the enablement of the entity.`sprite`'s child to the entity.`sprite`.enabled. This contradicts the goal of this behavior so what the behavior will do when it is active is to keep enforcing the rule that only one of the 2 can remain enabled by essentially fighting against EntityControl's Update.

Since this behavior is what keeps the `disguiseobj` from being disabled by EntityControl, whenever the player gets `inrange` and this behavior is switched away, it will no longer fight against EntityControl causing the entity.`sprite` to be enabled with the `disguiseobj` disabled. This is how an NPCControl with this behavior can reveal itself.

If the player gets out of `inrange` after this and we come back to this behavior, it now manages a cooldown (the `disguisecooldown`) of 120 frames that will have the entity move to its `startpos` when it gets to 20 frames left. From there, either the 20 frames expires or it reached the entity.`startpos` within 2.0 units which will cause this behavior to flip flop again such that entity.`sprite` is disabled and `disguiseobj` enabled. This puts back the disguise and the cycle continues.

Due to this complex management of the `disguiseobj`, this behavior's logic is very spread out throughout the game because it needs to fight against EntityControl.Update as there is very little logic there to support the disguise feature.

## Start / SetUp
On SetUp:
- entity.`alwaysactive` is set to true
- `disguiseobj` is initialised to the transform of `prefabs/Disguises/x` prefab where x is the entity.[AnimID](../../Enums%20and%20IDs/AnimIDs.md). It gets childed to the entity.`sprite` which also gets disabled making only the disguise visible instead of the sprite. The local position is ensured to be Vector3.zero, but the angles and the scales depends on the AnimID (anything not mentioned is left default):
  - `Cactus`: scale of (70.0, 25.0, 70.0)
  - `Mushroom`: scale of (20.0, 15.0, 25.0)
  - `CursedSkull`: scale of 0.5 uniform, local position of (0.0, 0.0, -0.5) and angles of (-90.0, 0.0, 0.0)
  - `Plumpling`: scale of 0.2 uniform and angles of (-90.0, 0.0, 180.0)

After SetUp returned on Start:
- `disguiseobj` is enabled
- `disguisecooldown` is set to -1 (meaning it simulates an already expired disguise cooldown)
- entity.`sprite` is disabled 
- enity.`height` is set to 0.0 

All of this hides the actual sprite of the entity with no visual offset and it leaves the disguise object visible.

## DoBehavior (Main logic)
Nothing happens if `disguisecooldown` is below 0 (meaning it expired more than a cycle ago).

The main logic depends on the `disguisecooldown` which is an int countdown of the number of update cycles left before the `disguiseobj` gets enabled and the entity.`sprite` disabled. This occurs if by the end of this cycle, it reaches 0 or below and the opposite happens if it not. No matter what specific branch of the logic applies, these enablement changes always happen after the branch is done.

### `disguisecooldown` is 0 (it just expired last cycle)
- TempSpin will be called on the entity which sets its `spin` to (0.0, 20.0, 0.0) and then zerored out in 0.2 seconds
- `disguisecooldown` is set to -1
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity

### `disguisecooldown` hasn't expired yet and either it's 20 or above or the NPCControl is 2.0 or less away from entity.`startpos`
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- `disguisecooldown` is decremented
- If `disguisecooldown` is 40 or 80, entity.`flip` is toggled (this is by default the 2/3 and 1/3 point of the cooldown)

### `disguisecooldown` hasn't expired yet and it's below 20 while the NPCControl is more than 2.0 away from entity.`startpos`
- [DetectDirection](../../EntityControl/EntityControl%20Methods.md#DetectDirection) is called on the entity with the entity.`startpos`
- Align the entity.`moverotater` to the entity.`startpos` via a LookAt call
- If a [HasGroundAhead](../../EntityControl/EntityControl%20Methods.md#hasgroundahead) call returns true, [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity to move the entity to its `startpos` ignoring the y component at normal speed with the state being `Walk` and the stopstate being `Idle`. Otherwise, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity setting the [animstate](../../EntityControl/Animations/animstate.md) to its existing one at the end
- The entity.[animstate](../../EntityControl/Animations/animstate.md) is set to the entity.`walkstate`
- The entity.`oldstate` is set to -1 (which forces a sprite update)
- If entity.`hitwall` is true:
 - The NPCControl position is set to entity.`startpos`
 - entity.`rigid` velocity is zeroed out
 - DeathSmoke particles are played at the NPCControl position if the entity is `incamera`

## Update (Inactive)
If the `disguisecooldown` is fully expired (it's -1):
- `disguiseobj` is enabled
- The entity.`sprite` gets disabled if entity.`campos` z is above 5.0 (meaning it's too far forward from the camera), it remains enabled otherwise
- `overrideminheight` is set to true
- The entity.`height` is set to 0.0

## LateUpdate (Not a `dummy` and the entity is `incamera`)
If the `freezecooldown` and `dizzytime` expired while we aren't `minipause`, the enity.`height` is set to a lerp from the existing one to `initialheight` or 0.0 with a factor of 1/10 of the game's frametime. What determines the destination height is if the `disguisecooldown` is 0 or above where it's `initialheight`, 0.0 otherwise.

Otherwise, if we are in a `minipause`, then the `disguiseobj` is enabled if `disguisecooldown` is -1 (fully expired), disabled otherwise while the opposite is done on the entity.`sprite`. The enablement values only changes when needed instead of every cycle by checking if the `disguiseobj` is enabled or not in comparison to `disguisecooldown` being -1 or not.

## Update (Frozen `Enemy` override)
- `disguisecooldown` is set to 120
- `disguiseobj` is disabled
- entity.`sprite` being enabled.

## Update (dizzy `Enemy` override)
The `disguiseobj` is disabled.

## [Dizzy](../Notable%20methods/Dizzy.md)
The `disguiseobj` is disabled with the entity.`sprite` being enabled.

## EntityControl.[Update](../../EntityControl/Update%20process/Unity%20events/Update.md#update) (active)
This behavior prevents the `sprite` from being enabled there. It also goes against this behavior at least as far as the `disguiseobj` is concerned, but this behavior doesn't change this logic in particular, only overrides it.

## EntityControl.[UpdateSprite](../../EntityControl/Update%20process/UpdateSprite.md#updatesprite)
This update method is disallowed to change the `sprite` enablement on an [Enemy](../NPCType.md#enemy) with this behavior.

## MainManager.RefreshEntities
This logic only matters if the method was called as a result of MapControl.Start, MapControl.RefreshInsides and BattleControl.ReturnToOverworld:
- `disguisecooldown` is set to -1 (fully expired)
- `disguiseobj` is disabled
- entity.`sprite` being enabled.

## MapControl.PreloadSprites
This behavior prevents the preloading of the entity.`sprite` texture into the map's `entitysprite`.

## [RespawnEnemy](../Notable%20methods/RespawnEnemy.md) (Only if it's the default behavior)
- The `disguiseobj` is destroyed if it was present
- The entity.`speed` is set to 2.0
- The `disguisecooldown` is set to -1
- The default behavior is set to [Wander](Wander.md)
- The entity.`onground` is set to true in 0.5 seconds

This will get further handled by DoBehavior, see the section below for details

## DoBehavior (behavior no longer exists)
A special case is handled where this behavior no longer exists, but the `disguiseobj` still does. The only way for this case to happen is if a destroy of the `disguiseobj` was requested by [RespawnEnemy](../Notable%20methods/RespawnEnemy.md) because not only it destroys it, but it also changes the default behavior to [Wander](Wander.md). If this was the previous one and it wasn't the `inrange` one, then it no longer exists after the respawn which makes it fall into this special case. The handling goes as follow:
- TempSpin is called on the entity which sets its `spin` to (0.0, 20.0, 0.0) and then zerored out in 0.2 seconds
- The `disguisecooldown` is set to 120.0
- The entity.`overridehright` is set to false
- The `disguiseobj` is disabled

Because the behavior was also changed, the main logic won't occur.