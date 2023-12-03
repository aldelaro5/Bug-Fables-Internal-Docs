# Enemy
An enemy is a non interactable entity that supports [ActionBehaviors](ActionBehaviors.md) with the ability to initiate a battle encounter with the player consisting of up to 4 [enemy](../../Enums%20and%20IDs/Enemies.md). It also features special logic to handle all of this including cases where it's frozen or dizzy. It is the NPCType with the most exclusive logic available.

## Data arrays
- `battleids`: The array of [enemy](../../Enums%20and%20IDs/Enemies.md) ids this battle encounter consist of

## Additional data
- `eventid`: If it's not 0, the `respawntimer` value whenever the enemy entity.`iskill` goes to true

## Start / SetUp
The `arrow` is initialised to a new HelpArrow parented to the NPCControl of color cyan with offset (0.75, 0.75, 0.75) with a distance of the entity.`freezesize.x` * 2.0 clamped from 2.5 to the entity.`freezesize.x` * 3.5. This prepares the help arrow to render should the enemy gets frozen. Its `lockcooldown` is set to true on Update as long as `freezecooldown` is above 0 (the enemy is frozen).

If the `eventid` is above 0 (we have a `respawntimer` feature), the entity.`alwaysactive` is set to true.

## Update (Non `dummy`)
The `lockcooldown` of the `arrow` is set to true only if `freezecooldown` is above 0.

## Update (Active, main logic, overrides)
Only 1 of 3 cases happens here (checked in order):
- This is a frozen enemy (the `freezecooldown` hasn't expired yet)
- This is a frozen entity of any kind (it has an `icecube`, but in practice, this can only happen if an enemy is frozen because it's only on them that [Freeze](../EntityControl/Notable%20methods/Freeze%20handling.md#Freeze) is called)
- This is a dizzy enemy (the `dizzytime` is above 0.0)

Additionally, there's logic that happens as part of a standard active update exclusive to an enemy if none of the cases above applied and we aren't `trapped`.

### Frozen enemy
If the entity.`rigid` y velocity is not between -0.1 and 0.1 inclusive, its x and z components are set to the `icevel` ones. Otherwise, `icevel` is set to Vector3.zero.

The absolute position is clamped to be within a radius of `radiuslimit` where the center is at the entity.`startpos` ignoring the y component.

The following occurs:
- The `pusher` is disabled if it was present
- The `dizzytime` is set to -1000.0
- The entity.`spin` is zeroed out
- The `disguisecooldown` is set to 120.0 and if a `disguiseobj` is present, it is disabled and the entity.`sprite` is disabled
- The entity.`emotioncooldown` is set to -1.0
- `templayer` is set to the current layer if it was negative. This is done because this also causes the current layer to be set to 8 (Ground)
- If the entity doesn't have an `icecube`, [Freeze](../EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on the entity. Otherwise, all collisions between the `scol` and the entity.`icecube` are ignored.
- If the `boxcol` isn't present, it is created non trigger with a size of the entity's `freezesize`
- The center of the `boxcol` is set to the entity's `freezeoffset`
- The `ccol` gets disabled
- The `boxcol` gets enabled
- The entity.`height` gets decreased by the game's frametime * 0.075 then clamped from 0.0 to 999.0
- If the `icecooldown` is below 100.0, the entity.`shakeice` is set to the invert of the entity.`inice`. Otherwise, the entity.`shakeice` is set to false
- If the entity is `onground` and not `inice` and we aren't in an `icemap`, the `freezecooldown` gets decreased by framestep
- If the entity is `onground`, the `freezeaircooldown` is set to 0.0. Otherwise, it is decreased by framestep on top of the `freezecooldown` being set to 0.0 if the `freezeaircooldown` was over 300.0
- entity.`animspeed` is seto to 0.0 and entity.`overrideanim` set to true
- If entity.`hasshadow` is true, the entity.`shadow` is disabled

### Frozen entity about to be unfrozen
It is assumed here that it's an enemy ready to be unfrozen, but this actually checks if entity.`icecube` exists to detect this.

If the player is present and it is `standingon` this `boxcol`, then the player.entity.`onground` is set to false and its `standingon` to null. This means if the player was standing on the ice block, the game forces the ground detector and its `standingon` field to not report to be on it.

[BreakIce](../EntityControl/EntityControl%20Methods.md#breakice) is called on the entity.

The layer is set to `templayer` if it wasn't -1 before it is set to -1. This restores the value it had when the entity was frozen previously.

The following occurs:
- entity.`shadow` is enabled if `hasshadow` was true
- entity.`ccol` is enabled
- entity.`boxcol` is disabled if it was present
- entity.`overrideanim` is set to false
- entity.`oldstate` is set to -1
- entity.`oldfly` is set from the opposite of `flyinganim`
- entity.`behaviorroutine` is in progress, then [SetAnim](../EntityControl/Animations/SetAnim.md) is called with force to true without arguments or `f` if the NPCControl `returntoheight` is true

### Dizzy enemy
The `disguiseobj` is disabled if present.

[StartBattle](StartBattle.md) is called if all of the following are true:
- It's been more than 20 frames since the `startlife`
- We aren't in a `pause`, `minipause` or [message](../../SetText/Notable%20states.md#message)
- The player is present and the distance between it and this enemey is \<= the entity.`ccol` radius + 1.1

The following occurs:
- entity.`height` is decreased by - 0.075 clamped from 0.0 to 999.0
- entity.`spin` is set to (0, `dizzytime` / 5 clamped from 0.0 to 15.0, 0.0)
- entity.`overrideanim` is set to true
- The entity.[animstate](../EntityControl/Animations/animstate.md) is set to 11 (Hurt)
- The entity.`emoticonid` is set to 1 (? emoticon) with an `emoticoncooldown` set to 10.0
- [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity if the entity is `onground` and the `touchcooldown` had expired
- `dizzytime` is decreased according to the game's frametime.
- The position is clamped to be within the radius `radiuslimit` where the center is the entity.`startpos` without consideration for the y component

## Update (Active, main logic, no overrides)
These logic can happen as part of a standard active Update if none of the above cases applied and we aren't `trapped`.

If the `dizzytime` is above -999.0, it is set to -1000.0 with the following adjustements being done on the entity which essentially ends the dizzyness:
- enity.`spin` gets zeroed out
- enity.[animstate](../EntityControl/Animations/animstate.md) gets set back to entity.`basestate`
- enity.`overrideanim` is set to false

[StartBattle](StartBattle.md) is called if all of the following are true:
- It's been more than 20 frames since the `startlife`
- We aren't in a `pause`, `minipause` or [message](../../SetText/Notable%20states.md#message)
- The player is present and the distance between it and this enemey is \<= the entity.`ccol` radius + 1.1

## Update (Inactive)
For every 3 frames, if the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove), [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity except if it has a [SetPath](ActionBehaviors/SetPath.md) or [StealhAI](ActionBehaviors/StealthAI.md) behavior

## LateUpdate (Non `dummy` and entity is `incamera`)
If the `eventid` is 0 and the entity `iskill` while we aren't in `pause`, `minipause` and `inevent`, some logic occurs depending on the `respawntimer`:
- If it is -100.0 or less, it is set to `eventid`,
- If it is above 0.0, it is decreased by the game's frametime
- If it's negative, but above -100.0, [RespawnEnemy](RespawnEnemy.md) is called using the entity `spawnpoint`, the entity `iskill` is set to false and `respawntimer` is set to -110.0

## LateUpdate (Every 3 frames during RefreshPlayer)
When the `inrange` value changes:
- If the new `inrange` is true, the change is only accepted if both the `freezecooldown` and the `dizzytime` expired. If they have, the `Find` sound is played if the player wasn't `digging`
- If the new `inrange` is false and both `freezecooldown` and `dizzytime` expired while `behaviorroutine` isn't in progress:
  - entity.`emoticonid` is set to 1 (?)
  - entity.`emoticoncooldown` is set to 70.0
  - [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called with the `Idle` target state
  - The `Lost` sound is played
  - entity.`overrideonlyflip` is set to false

## OnTriggerEnter
What happens here depends on the other collider, only one branches is ran and if it doesn't apply, nothing happens.

### The other tag is `Icecle`
This does nothing if the entity is `digging` or its [animid](../../Enums%20and%20IDs/AnimIDs.md) is `FireKrawler`, `FireCape`, `FireWarden` or `Strider`.

- The other gameObject is destroyed if it had a RigidBody
- The `freezecooldown` is set to 300.0 with some exceptions:
  - It is set to 30.0 if the current [map](../../Enums%20and%20IDs/Maps.md) is `GiantLairDeadLands1` or `GiantLairDeadLands2`
  - It is set to `freezetime` clamped from 600.0 to infinity if the Extra Freeze [medal](../../Enums%20and%20IDs/Medal.md) is equipped or `extrafreeze` is true
- If it's a `ToeBiter` and `internaltransform[0]` (his boulder) is present, it is destroyed
- entity.`rigid` x and z velocity are zeroed out
- entity.`onground` is set to true
- [StopForceBehavior](StopForceBehavior.md) is called

### The other tag is `BeetleDash` or `BeetleHorn`
The `BeetleHorn` tag only counts here if the player isn't `dashing`. This does nothing if `collisionammount` is 0, the entity is `dead` or `digging` or the `touchcooldown` hasn't expired yet.

- TempIgnoreColision is called on the entity with the other collider which ignores all collision between the entity.`ccol` / entity.`detect` and the other collider for 0.5 seconds
- The `Damage0` sound is played
- If the other collider tag was `BeetleDash`, the `freezecooldown` is set to 0.0
- A [Dizzy](Dizzy.md) coroutine is started for 120.0 frames with force launch with the direction being the normalized direction from this enemy to the player * 5.0
- `collisionammount` is incremented

### The other tag is `BeeRang`
This does nothing if `collisionammount` is 0, the entity is `dead` or `digging` or the `touchcooldown` hasn't expired yet.

- The `WoodHit` sound is played on the entity
- A [Dizzy](Dizzy.md) coroutine is started for 80.0 frames with no pushforce or force launch
- `collisionammount` is incremented

### The other collider is owned by the player
If we aren't in a `minipause`, `pause` or `inevent`, [StartBattle](StartBattle.md) is called TODO: this seems contradictory with the Update logic ???.
