# LateUpdate
The logic depends on the [NPCType](NPCType.md), [ObjectTypes](Object.md#objecttypes), [ActionBehaviors](ActionBehaviors.md) and [Interaction](Interaction.md). Consult each's documentation to learn more.

There are mainly 3 sections to this LateUpdate:

- A handler to a NaN position
- Cleaning the `destroyOnBattle` list
- The main logic when it's not a `dummy`

## NaN position handling
This section applies if any of the component of the position is a NaN and the entity.`startpos` is not null.

If we got here, it means that the entity somehow got so far out of bounds that the components no longer reports valid floating point numbers. This is handled by setting the position to the entity.`startpos` and setting the entity.`onground` to false.

Some specific logic happens if we are `trapped` related to the [CoiledObject](ObjectTypes/CoiledObject.md) object type.

## Cleaning the `destroyOnBattle` list
The only thing that happens in this section is if `destroyOnBattle` has more than 10 elements, all the null ones are removed. This can happen when a `ToeBiter`'s [ShootProjectile](ActionBehaviors/ShootProjectile.md) causes a lot of projectiles to be added to the list, but got destroyed after landing which made their reference null. Since it's not useful to keep null references as they were already destroyed, this section removes them when the list grows to have more than 10 elements.

## Non Dummy logic
This section starts with the [NPC](NPC.md#npc) NPCType exclusive logic involving the `insideid` and whether or not the entity.`rigid` should be locked or unlocked. Consult the corresponding NPCType [section](NPC.md#lateupdate-non-dummy) to learn more.

After this whole logic, the rest of this section only happens if the entity is `incamera`. From there, there are several logic that occurs conditionally:

- The `pusher` enablement and center are updated when applicable, see the [NPCType enum table](NPCType.md#enum-table) to learn more.
- If `startlife` is less than 300.0, it is incremented by framestep.
- Some [WanderOffscreen](ActionBehaviors/WanderOffscreen.md) exclusive logic occurs here when applicable
- If `dirtcd` hasn't expired yet, it is decreased by framestep
- Unles it's an [Object](Object.md), the far fading logic occurs. Check the [NPCType enum table](NPCType.md#enum-table) to learn more.
- If `interactcd` hasn't expired yet, it is decreased by framestep
- Some [StorageAnt](Interaction/StorageAnt.md) exclusive logic occurs here when applicable
- Some [CoiledObject](ObjectTypes/CoiledObject.md) exclusive logic occurs here when applicable
- Some [Enemy](Enemy.md) exclusive logic occurs here when applicable
- Some `disguiseobj` related [ActionBehaviors](ActionBehaviors.md) occurs here when applicable
- Every 3 frames, some logic is performed, see the section below
- If the `behaviorcooldown` hasn't expired yet, it is decreased by the game's frametime
- The `tattleid` is set to a value if there's a specific [Interaction](Interaction.md).
- Some [Shop](Interaction/Shop.md) and [CaravanBadge](Interaction/CaravanBadge.md) exclusive logic occurs here when applicable
- Some [Geizer](ObjectTypes/Geizer.md) exclusive logic occurs here when applicable
- If the entity `iskill` is false:
    - If the y position is less than the map.`ylimit`, then the position is set to the entity.`startpos`. On top of this, DeathSmoke particles are played at the entity.`sprite` position if the entity is `incamera`, it isn't `dead` and the [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't negative (it isn't `None`). An exception to this logic is if it's a [Dropplet](ObjectTypes/Dropplet.md) object
    - Some [WanderOnWater](ActionBehaviors/WanderOnWater.md) and [ChaseOnWater](ActionBehaviors/ChaseOnWater.md) exclusive logic occurs here when applicable

### Every 3 frames
RefreshPlayer (see the section below for details) is called if the entity is `incamera` or the map.`limitbehavior` is false and all of the following are true:

- We aren't `inevent`
- We aren't in a `minipause`
- The `insideid` matches the current one
- The entity isn't `dead`, `iskill` and `deathcoroutine` isn't in progress

This also manages the player `npc` list if this isn't the player [beemerang](ObjectTypes/Beemerang.md). This NPCControl is added to the list if `inrange` is true and it wasn't in it and it is removed from the list if `inrange` is false and it was in it.

If this is an [NPC](NPC.md), the emotes system updates occurs here, check the [NPCType enum table](NPCType.md#enum-table) to learn more.

The entity.`ccol` height is set to `colliderheight` and its center is set to (0.0, half of `colliderheight`, 0.0) if this isn't the player's [beemerang](ObjectTypes/Beemerang.md) and the entity.`ccol` height isn't exactly the `colliderheight`.

If the entity `iskill` and the y position is above -999.0:

- The entity.`rigid` has its gravity disabled
- The entity.`ccol` is disabled
- The `boxcol` is disabled if it is present
- The position is set to (0.0, -1000.0, 0.0) which is offscreen

#### RefreshPlayer
This is a private method only used in LateUpdate.

Its job is to update the `inrange` value and it receives it in parameter. The new value is true if the distance between the player and this is less than the radius and the `insideid` matches the current one, false otherwise. The new value is computed, but not yet changed because if it's an [Enemy](Enemy.md), there's some override cases where the change will be denied. Enemies also have specific logic exclusive to them in this method, consult the corresponding NPCType [section](Enemy.md#lateupdate-every-3-frames-during-refreshplayer) to learn more.

If the value change is allowed, it is performed followed by some [StealthAI](ActionBehaviors/StealthAI.md) exclusive logic when applicable.

## End
No matter which sections applied, the `collisionamount` is set to 0 and `ignoreconstraint` to false.