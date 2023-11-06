# NPCType
This is an enum that dictates the main capabilities of an NPCControl using the `entitytype` field of that type. Each value offers wildly different behaviors with one being reserved specifically for shop logic. It is the most impactful field on the lifecycle of an NPCControl.

## Enum table
The following tables includes different columns to better illustrate the main properties differences between the different enum value:
- Tag: The gameObject tag assigned on Start
- Behaviors?: Whether or not [ActionBehaviors](ActionBehaviors.md) are enabled
- Dialogues?: Whether or not [GetDialogue](Notable%20methods/GetDialogue.md) gets called which initialises a standard system to have the NPCControl call [SetText](../../SetText/SetText.md) conditionally with the lines coming from the `dialogues` field.
- Interactions?: Whether or not [Interactions](Interaction.md) are supported which allows the player (or the game via a methoid call) to interact with the NPCControl in various ways.
- Emotes?: Whether or not [CheckEmoteFlag](Notable%20methods/CheckEmoteFlag.md) gets called during LateUpdate every 3 frames when `canpause` is true and the first NPCControl in the player.`npc` list isn't this one (or the list is empty). This method initialises a standard emoticon system (this doesn't include manual changes the game does in specific situations)
- `ccol`: The state of entity.`ccol` which is the [EntityControl](../EntityControl/EntityControl.md) standard non trigger CapsuleCollider.
- `scol`: The state of the `scol` which is a trigger SphereCollider NPCControl manages.
- `pusher`: The state of the `pusher` which is a trigger CapsuleCollider with a height twice the `ccol` and a raidus of twice as narrow. This is specifically present to have the player be pushed away if they collide with it. Its enablement is updated in LateUpdate while `dummy` is false and entity.`incamera` is true which keeps it enabled as long as we aren't `inevent`, [message](../../SetText/Notable%20states.md#message) is released and we aren't in a `minipause`, it is disabled otherwise. LateUpdate also ensures its center is set to (0.0, `colliderheight` + the entity `height`, 0.0).
- `rigid` mass: The mass of the entity.`rigid`.
- Far fading?: This is some logic in LateUpdate when `dummy` is false and entity.`incamera` is true where it will slightly fade out NPCControl too far away from the camera and fade them back in when getting close enough again. The logic adjusts the alpha of the `sprite` so that it is a lerp from the existing one to either 1.0 (opaque) or 0.3 (30% visible) with a factor of a 1/10 of the game's frametime. This fades in the entity or fades it out and what determines this is the entity `compos.z`: if it's 2.5 or above, we are fading in and fading out otherwise. The specific logic happens if all of the following are true:
  - `startlife` is above 20.0
  - The entity [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't `None`
  - There is no `model`
  - The `sprite` is present
  - It's not a `hologram`
  - The current `insideid` matches this entity
TODO: interesting, but unsure if it works because EntityControl also manages this

|Value|Name|Tag|Behaviors?|Dialogues?|Interaction?|Emotes?|`ccol`|`scol`|`pusher`|`rigid` mass<sup>1</sup>|Far fading?|
|----:|----|---|----------|---------|------------|-------|------|------|--------|-----------------------|-----------|
|0|NPC|NPC|Yes<sup>5</sup>|Yes|Yes|Yes|Enabled|Disabled|Yes<sup>5</sup>|10000.0|Yes|
|1|Enemy|Enemy|Yes|No|No|No|Enabled<sup>2</sup>|Disabled<sup>2</sup>|No|100.0|Yes|
|2|Object|Object|No|No|No|No|disabled<sup>4</sup>|Enabled<sup>3</sup>|No|10000.0|No|
|3|SemiNPC|NPC|Yes|No|Yes|No|Enabled|Enabled<sup>3</sup>|No|10000.0|Yes|

1: The mass is 1.0 if the entity.`item` is true regardless of the type.

2: The `secondcoll` replaces it with the same attributes than the `ccol` and the player `ccol` and `detect` (its wall detector) ignores any enemy `ccol` meaning the player can only collide with the `secondcoll`, but other RigidBody can collide with both. Additionally, the player `detect` ignores the `secondcoll`.

3: The `scol` is disabled if entity.`item` is true. Also, it is left to null (not even added as a component) if it's The player [Beemerang](ObjectTypes/Beemerang.md).

4: Exceptions applies where the `ccol` remains enabled depending on `objecttype`:
- [SavePoint](ObjectTypes/SavePoint.md)
- [RollingRock](ObjectTypes/RollingRock.md)
- [FixedAnim](ObjectTypes/FixedAnim.md) (managed by a data field)
- [Item](ObjectTypes/Item.md)
- The player [Beemerang](ObjectTypes/Beemerang.md) (only remains enabled if the entity `fixedentity` is false)

5: These features are excluded for any of the following cases apply (Behaviors are always enabled, but [SetInitialBehavior](SetInitialBehavior.md) is not called under these cases):
- entity.`item` is true (TODO: what is an item NPC ???)
- entity.[animid](../../Enums%20and%20IDs/AnimIDs.md) is negative
- `interacttype` is [Shop](Interaction/Shop.md) or [CaravanBadge](Interaction/CaravanBadge.md)

While a lot of the logic of an NPCControl is shared regardless of its `entitytype`, the vast majority is dictated to a specific one that the others only partially have or do not have at all.

The following sections details the specific behaviors each has that isn't mentioned in the table above.

## NPC

### Start / SetUp
The `radius` is incremented by 0.35 if it was less than 1.75.

### Update (Inactive)
For every 3 frames, if the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove) without any `SetPath` or `StealhAI` behavior, [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity.

### LateUpdate (Non `dummy`)
We decides if we should lock/unlock the entity `rigid` and call [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) and it only when the entity isn't `dead`, `iskill`, `deathcoroutine` isn't in progress and `activeonpause` is false. It depends if we are `inevent` and the current `insideid`:
- If we aren't: 
  - if the `originalid` isn't -1 (it's defined), we ensure the entity `rigid` is locked via [LockRigid](../EntityControl/EntityControl%20Methods.md#lockrigid) if the current `insideid` doesn't match this `NPC` and unlocked if it does match. This is only performed when the current value of the entity `rigid.isKinematic` doesn't match the reality of the `insideid` matches or not.
  - If entity is in a `forcemove` and the current `insideid` doesn't match this `NPC`, [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called
- If we are, the entity isn't a `fixedentity`, the `originalid` isn't -1 (it's defined) and it's not in a `forcemove`, [LockRigid(false, false)](../EntityControl/EntityControl%20Methods.md#lockrigid) is called if the `insideid` matches this `NPC`, but if it doesn't, [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called instead

### OnTriggerEnter
Only 2 things are handled here: the `specialinteract` if it's not `None` and some handling for the [StealthAI](ActionBehaviors/StealthAI.md) behavior.

The `specialinteract` part does the following if the other collider tag is `BeetleDash` (or `BeetleHorn` if the `specialinteract` is `AnyHorn`):
- Set `interactedwithhit` to true
- Calls [Interact](Interact.md) with null arguments
- Set `interactedwithhit` to false in 1 second

As for the `StealthAI` part, check its documentation to learn more.

## Enemy

### Start / SetUp
The `arrow` is initialised to a new HelpArrow parented to the NPCControl of color cyan with offset (0.75, 0.75, 0.75) with a distance of the entity `freezesize.x` * 2.0 clamped from 2.5 to the entity `freezesize.x` * 3.5. This prepares the help arrow to render should the enemy gets frozen. Its `lockcooldown` is set to true on Update as long as `freezecooldown` is above 0 (the enemy is frozen).

If the `eventid` is above 0, the entity's `alwaysactive` is set to true TODO: what does this mean and why is this related to events ??? seems involved in LateUpdate ???

### Update (Non `dummy`)
The `lockcooldown` of the `arrow` is set to true only if `freezecooldown` is above 0.

### Update (Active, main logic, overrides)
Only 1 of 3 cases happens here (checked in order):
- This is a frozen `Enemy` (the `freezecooldown` hasn't expired yet)
- This is a frozen entity of any kind (it has an `icecube`, but in practice, this can only happen if an enemy is frozen because it's only on them that [Freeze](../EntityControl/Notable%20methods/Freeze%20handling.md#Freeze) is called)
- This is a dizzy `Enemy` (the `dizzytime` is above 0.0)

Additionally, there's logic that happens as part of a standard active update exclusive to an `Enemy` if none of the cases above applied and we aren't `trapped`.

#### Frozen enemy
First, if the entity's `rigid` y velocity is not between -0.1 and 0.1 inclusive, its x and z components are set to the `icevel` ones. Otherwise, `icevel` is set to Vector3.zero.

The absolute position is clamped to be within a radius of `radiuslimit` where the center is at the entity's `startpos` ignoring the y component.

The following occurs:
- The `pusher` is disabled if it was present
- The `dizzytime` is set to -1000.0
- The entity's `spin` is zeroed out
- The `disguisecooldown` is set to 120.0 and if a `disguiseobj` is present, it is disabled and the entity's `sprite` is disabled
- The entity's `emotioncooldown` is set to -1.0
- `templayer` is set to the current layer if it was negative. This is done because this also causes the current layer to be set to 8 (Ground)
- If the entity doesn't have an `icecube`, [Freeze](../EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on the entity. Otherwise, all collisions between the `scol` and the entity's `icecube` are ignored.
- If the `boxcol` isn't present, it is created non trigger with a size of the entity's `freezesize`
- The center of the `boxcol` is set to the entity's `freezeoffset`
- The `ccol` gets disabled
- The `boxcol` gets enabled
- The entity's `height` gets decreased by the game's frametime * 0.075 then clamped from 0.0 to 999.0
- If the `icecooldown` is below 100.0, the entity's `shakeice` is set to the invert of the entity's `inice`. Otherwise, the entity's `shakeice` is set to false
- If the entity is `onground` and not `inice` and we aren't in an `icemap`, the `freezecooldown` gets decreased by framestep
- If the entity's is `onground`, the `freezeaircooldown` is set to 0.0. Otherwise, it is decreased by framestep on top of the `freezecooldown` being set to 0.0 if the `freezeaircooldown` was over 300.0
- The entity's `animspeed` is seto to 0.0 and `overrideanim` set to true
- The the entity `hasshadow`, the entity's `shadow` is disabled

#### Frozen entity about to be unfrozen
It is assumed here that it's an `Enemy` ready to be unfrozen

First, if the player is present and it is `standingon` this `boxcol`, then the player's entity `onground` is set to false and its `standingon` to null. This means if the player was standing on the ice block, the game forces the ground detector and its `standingon` field to not report to be on it.

Then, [BreakIce](../EntityControl/EntityControl%20Methods.md#breakice) is called on the entity.

The layer is set to `templayer` if it wasn't -1 before it is set to -1. This restores the value it had when the entity was frozen previously.

A couple of adjustements happens on the entity:
- The `shadow` is enabled if `hasshadow` was true
- The `ccol` is enabled
- The `boxcol` is disabled if it was present
- The `overrideanim` is set to false
- The `oldstate` is set to -1
- The `oldfly` is set from the opposite of `flyinganim`
- If no `behaviorroutine` is in progress, then [SetAnim](../EntityControl/Animations/SetAnim.md) is called with force to true without arguments or `f` if the NPCControl `returntoheight` is true

#### Dizzy enemy
First, the `disguiseobj` is disabled if present.

[StartBattle](StartBattle.md) is called if all of the following are true:
- It's been more than 20 frames since the `startlife`
- We aren't in a `pause`, `minipause` or [message](../../SetText/Notable%20states.md#message)
- The player is present and the distance between it and this enemey is \<= the entity's `ccol.radius` + 1.1

Some adjustements happens on the entity:
- The `height` is decreased by - 0.075 clamped from 0.0 to 999.0
- The `spin` is seto to (0, `dizzytime` / 5 clamped from 0.0 to 15.0, 0.0)
- The `overrideanim` is set to true
- The [animstate](../EntityControl/Animations/animstate.md) is set to 11 (Hurt)
- The `emoticonid` is set to 1 (? emoticon) with an `emoticoncooldown` set to 10.0
- [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called if the entity is `onground` and the NPCControl `touchcooldown` had expired

Then, `dizzytime` is decreased according to the game's frametime.

Finally, the absolute position is clamped to be within the radius `radiuslimit` where the center is the entity's `startpos` without consideration for the y component.

### Update (Active, main logic, no overrides)
These logic can happen as part of a standard active Update if none of the above cases applied and we aren't `trapped`.

If the `dizzytime` is above -999.0, it is set to -1000.0 with the following adjustements being done on the entity which essentially ends the dizzyness:
- The `spin` gets zeroed out
- The [animstate](../EntityControl/Animations/animstate.md) gets set back to the `basestate`
- The `overrideanim` is set to false

[StartBattle](StartBattle.md) is called if all of the following are true:
- It's been more than 20 frames since the `startlife`
- We aren't in a `pause`, `minipause` or [message](../../SetText/Notable%20states.md#message)
- The player is present and the distance between it and this enemey is \<= the entity's `ccol.radius` + 1.1

### Update (Inactive)
For every 3 frames, if the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove), [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity except if it has a [SetPath](ActionBehaviors/SetPath.md) or [StealhAI](ActionBehaviors/StealthAI.md) behavior

### LateUpdate (Non `dummy` and entity is `incamera`)
If the `eventid` is 0 and the entity `iskill` while we aren't in `pause`, `minipause` and `inevent`, some logic occurs depending on the `respawntimer`:
- If it is -100.0 or less, it is set to `eventid`,
- If it is above 0.0, it is decreased by the game's frametime
- If it's negative, but above -100.0, [RespawnEnemy](RespawnEnemy.md) is called using the entity `spawnpoint`, the entity `iskill` is set to false and `respawntimer` is set to -110.0

### LateUpdate (Every 3 frames during RefreshPlayer)
When the `inrange` value changes:
- If the new `inrange` is true, there is an early return if the `freezecooldown` or `dizzytime` hasn't expired yet which cancels the `inrange` change and the StealthSpot call. If they have, the `Find` sound is played if the player wasn't `digging`
- If the new `inrange` is false and both `freezecooldown` and `dizzytime` expired while `behaviorroutine` isn't in progress, adjustements are done on the entity:
  - `emoticonid` is set to 1 (?)
  - `emoticoncooldown` is set to 70.0
  - [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called with the `Idle` target state
  - The `Lost` sound is played
  - `overrideonlyflip` is set to false

### OnTriggerEnter
What happens here depends on the other collider, only one branches is ran and if it doesn't apply, nothing happens.

#### The other tag is `Icecle`
If the entity isn't `digging` and its [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't `FireKrawler`, `FireCape`, `FireWarden` or `Strider` the following occurs:
- The other gameObject is destroyed if it had a RigidBody
- The freezecooldown is set to 300.0 with some exceptions:
  - It is set to 30.0 if the current [map](../../Enums%20and%20IDs/Maps.md) is `GiantLairDeadLands1` or `GiantLairDeadLands2`
  - It is set to `freezetime` clamped from 600.0 to infinity if the Extra Freeze [medal](../../Enums%20and%20IDs/Medal.md) is equipped or `extrafreeze` is true
- If it's a `ToeBiter` and `internaltransform[0]` (his boulder) is present, it is destroyed
- The entity `rigid` x and z velocity are zeroed out
- The entity `onground` is set to true
- [StopForceBehavior](StopForceBehavior.md) is called

#### The other tag is `BeetleDash` or `BeetleHorn`
The `BeetleHorn` tag only counts here if the player isn't `dashing`.

If `collisionammount` is 0, the entity is not `dead` or `digging` and the `touchcooldown` expired:
- TempIgnoreColision is called on the entity with the other collider which ignores all collision between the entity `ccol` / `detect` and the other collider for 0.5 seconds
- The `Damage0` sound is played
- If the other collider tag was `BeetleDash`, the `freezecooldown` is set to 0.0
- A [Dizzy](Dizzy.md) coroutine is started for 120.0 frames with force launch with the direction being the normalized direction from this `Enemy` to the player * 5.0
- `collisionammount` is incremented

#### The other tag is `BeeRang`
If `collisionammount` is 0, the entity is not `dead` or `digging` and the `touchcooldown` expired:
- The `WoodHit` sound is played on the entity
- A [Dizzy](Dizzy.md) coroutine is started for 80.0 frames with no pushforce or force launch
- `collisionammount` is incremented

#### The other collider is owned by the player
If we aren't in a `minipause`, `pause` or `inevent`, [StartBattle](StartBattle.md) is called TODO: this seems contradictory with the Update logic ???.

## Object
Most of the logic depends entirely on the `objecttype` field. For more information, consult the documentation about its type, [ObjectTypes](ObjectTypes.md).

There are a couple of special logic to mention:
- In OnTriggerEnter, if the other collider isn't the player, `collisionammount` is incremented.
- This is the only type with logic in OnTriggerStay and OnTriggerExit for specific `objecttype`

## SemiNPC
Most of the logic depends entirely on the `objecttype` field. For more information, consult the documentation about its type, [ObjectTypes](ObjectTypes.md).

One thing to mention, OnTriggerEnter is completely excluded from this type and it has no logic in there.

TODO