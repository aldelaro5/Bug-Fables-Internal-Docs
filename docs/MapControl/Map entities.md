# Map entities
It's possible to configure the behaviors of all [NPCControl](../Entities/NPCControl/NPCControl.md) on the map at once.

It involves the following configuration fields:

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|ylimit|float|From prefab (may be overriden by Hazard's Start)|A lower bound limit of the y position of entities in the map. If a player or map entity's y position gets lower than this value, they will respawn. The value is overriden to -150.0 if there's a Hazard present with a `type` of `Hole` due to its Start logic|-50.0|
|icemap|bool|From prefab|If this is true, the entire map is treated as an ice map meaning every entity's Start marks their `inice` to true which has several ice related changes|false|
|limitbehavior|bool|From prefab|If this is true, it restricts more what is considered an active NPCControl. For Update, it changes the maximum z `campos` allowed before becoming inactive from 25.0 to 15.0. For LateUpdate, it mandates that the entity is `incamera` for `inrange` updates to happen|false|
|keepobjectsactive|bool|From prefab|If this is true, LateUpdate won't update the enablement of too far away NPCControl and disable their `emoticon` every 2 frames (after `alivetime` reached 0.0)|false|

## `ylimit`
This field allows to essentially set the lowest y position NPCControl are allowed to be before they get respawned. This doesn't apply if there's an Hazard of type `Hole` since the field gets overriden to -150.0.

It's mainly intended to be an anti out of bounds measure because while it shouldn't be possible, it can happen under specific circumstances for an NPCControl to clip through the floor and fall. This feature would catch this problem and respawn the entity.

## `icemap`
Some maps are considered to be perpetually cold which means all the entity's `inice` will be set to true on their EntityControl's Start.

This will have the following effects on entities:

- On an [enemy](../Entities/NPCControl/Enemy.md) NPCControl, all [Krawler](../Battle%20system/Enemy%20actions/Enemies/Krawler.md) or [CursedSkull](../Battle%20system/Enemy%20actions/Enemies/CursedSkull.md), included in the encounter will have the following happen to them on [StartBattle](../Battle%20system/StartBattle.md):
    - `freezeres` increased by 70
    - Their `inice` set to true
    - [weakness](../Battle%20system/Actors%20states/Enemy%20features.md#weakness) set to only contain `HornExtraDamage`
    - Their actions include different behaviors, check their actions documentation to learn more
- All `Krawler` or `CursedSkull]` [animids](../Enums%20and%20IDs/AnimIDs.md) features different [AnimSpecificQuirks](../Entities/EntityControl/Animations/AnimSpecific.md#animspecificquirks) and [CheckSpecialID](../Entities/EntityControl/Notable%20methods/CheckSpecialID.md) logic
- All entities which has their `hasiceanim` set to true from their [endata](../TextAsset%20Data/Entity%20data.md) will have [SetAnim](../Entities/EntityControl/Animations/SetAnim.md) append the `i` argument
- No [enemy](../Entities/NPCControl/Enemy.md) NPCControl will be able to thaw out after being frozen
- All [PushRock](../Entities/NPCControl/ObjectTypes/PushRock.md) configured as slidable ice cubes will be allowed to move

## `limitbehavior`
This field can be seen as a more restrictive logic when it comes to consider which NPCControl is considered active.

It's intended as an opt-in optimisations to maps that needs to define a lot of entities which can require more processing. Here's what it changes on NPCControl:

- [Update](../Entities/NPCControl/Update.md): it changes the maximum z `campos` allowed before becoming inactive from 25.0 to 15.0 which means entities located further away from the camera won't be considered active
- [LateUpdate](../Entities/NPCControl/LateUpdate.md): it mandates that the entity is `incamera` for `inrange` updates to happen which is something managed by EntityControl that determines if the entity is in the bounds of the camera

## `keepobjectsactive`
This field only starts to apply once `alivetime` becomes 0.0 or below and only every 2 frames when `player` exists.

On LateUpdate, MapControl normally goes over all [NPC](../Entities/NPCControl/NPC.md) NPCControl and does a xz distance check between them and the `player`. It would then update their enabelement depending if the player is closer or further away than the NPC's `radious` * 2.0. The NPC will be enabled if it's closer and disabled otherwise. If the enablement changes, their `emoticon` gets disabled except if their [Interaction](../Entities/NPCControl/Interaction.md) is `Talk`.

However, `keepobjectsactive` being true on the map changes this: it would no longer implicate this logic. Under normal gameplay, only the `CaveOfTrials` [map](../Enums%20and%20IDs/Maps.md) has this field set to true.
