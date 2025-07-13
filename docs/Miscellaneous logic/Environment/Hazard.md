# Hazards
This component can be attached to a GameObject via Unity or added programmatically, but it requires a BoxCollider component that this component will force to be trigger.

A Hazards is considered a dangerous zone in a [map](../../Enums%20and%20IDs/Maps.md) and any contact with it will be handled depending on the hazard type and the specifics of what is trigger colliding with it. The vast majority of them are defined in Unity in maps, but a [GlowTrigger](../Rendering%20components.md#glowtrigger) can add its own if their `electime` isn't 0.0. All collision handling is done on OnTriggerEnter with some logic on LateUpdate, Update and Start.

## Configuration fields
There are various configurations available for hazards, here are all the public fields:

- `type`: An enum value that represents the type of the Hazards which greatly influences the logic. See the section below about the possible values and how they change in logic
- `centerpoint`: For a `SandFunnel` hazard, this is the point at which HazardAction will move the player towards when they collide with the Hazard. For a `Water` hazard, only the y component is used and it represents the base y coordinate used to move the `waterfloats` when their `flagfloats` requirements are met
- `riverammount`: The direction and magnitude per frame at which icicle platforms (created by `Moth`'s [Icicle](../../PlayerControl/Field%20abilities.md#icicle) ability) moves which includes the `parent` of any GroundDetector
- `speed`: The amount of frames it takes for a `TempFire` hazard to toggle its fire state (between on or off)
- `holdspace`: A y coordonate to use for temporarilly placing an entity before returning it or respawning it. It only applies for returning non `PFollower` tagged entities childed to the MainManager.`map` that aren't items created from [CreateItem](../../Entities/EntityControl/EntityControl%20Methods.md#createitem) and it also applies for player collision with a `Water`, `Honey` or `Hole` hazards
- `minwaterfloat`: The minimum y coordinate that any `waterfloats` can be on when their `flagfloats` requirements are met
- `freezetime`: The amount of frames an active icicle platform (created from `Moth`'s Icicle ability) that can ellapse before the platform disappears and becomes inactive. If this is 0.0 or below, the value defaults to 1000.0, but if the `FreezeTime` medal is equipped, this defaults to 2000.0. NOTE: Whether or not the medal is equipped does nothing if the value is positive
- `noice`: If this is true, it disables the ability for `Moth`'s Icicle ability to create icicle platforms and any attempt to create one will cause the icicle to shatter
- `keeplayer`: If this is true, the layer of the GameObject won't be set to 0 (`Default`) on Start so the value set initially is kept
- `alwaysactive`: If this is true, it allows for a `TempFire` hazard's `speed` timer logic to continue on Update and for a `Water` type hazard to move any icicle platform logic on Update even if the game is paused
- `iceparentedtomap`: If this is true, all icicle platforms used for this hazard are childed to the MainManager.`map` instead of the attached GameObject
- `playeronly`: If this is true, any entity other than the player and `PFollower` tagged entities won't cause a reaction when colliding with the hazard. If this is false, this only happens when in a `minipause` for any entity other than the player
- `platformlimit`: The maximum amount of icicle platforms allowed on this hazard. If the value is higher than 3, it is set to 3 on Start
- `waterfloats`: For a `Water` hazard, an array of all the GameObject whose matching `flagfloats` requirements are met will be allowed to float over the water by having their y position continuously lerp towards `centerpoint`.y clamped from `minwaterfloat`. On Start, all the GameObjects in this array have their isStatic set to false
- `flagfloats`: An array where each element corresponds to a flags slot where if it is true, it will allow the matching `waterfloats` to have their y continously lerped towards `centerpoint`.y calmped from `minwaterfloat`
- `respawnentities`: An array of entity ids whose entities's positions will be set to their `startpos` when HazardAction happens (when the player collides with a hazard leading them to be respawned)

## Hazard types
Here are the different hazard types and their specific logic:

- `Spikes`: Spikey floor that can't be walked on even if the player is shielded:
    - The HazardAction animation when the player collides with the hazard is the `Damage0` sound followed by them spinning for 0.75 seconds
    - Any entity other than the player colliding with the hazard will be returned, but any [NPCControl](../../Entities/NPCControl/NPCControl.md) will have their `freezecooldown` set to 0.0 which will unfreeze them
- `SandFunnel`: A funnel that sucks in from the ground
    - The HazardAction animation when the player collides with the hazard is the `SandFall` sound followed by them spinning and moving towards `centerpoint` until the MainManager.GetDistance between the 2 is 0.5 or lower
    - If the entity that collides with the hazard follows someone, they will not be returned
- `Water`: A body of water
    - On Start, MainManager.`map`.`lastwater` is set to the attached GameObject which can control how [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)'s sprout renders
    - Icicle platforms are allowed to exist on this hazard when an `IceFall` tagged collider collides with the hazard (only once per frame)
    - Any `waterfloats` GameObject whose `flagfloats` requirement is satisfied will have their y position moved via lerps to `centerpoint`.y clamped from `minwaterfloat`
    - Any entity colliding with the hazard will instantiate a `Prefabs/Objects/WaterSplash` prefab at the collider's GameObject and be destroyed in 0.75 seconds. Also, any collider tagged `DelAftBtl` will have the same happen if the game isn't paused, but it will also cause the collider's GameObject to be moved offscreen at (2222.0, -99999.0, 88282.0)
    - The HazardAction animation when the player collides with the hazard is player y position being set to `holdspace` 
    - Any entity other than the player colliding with the hazard will be returned, but any NPCControl will have their `freezecooldown` set to 0.0 which will unfreeze them
- `Hole`: A bottomless pit
    - On Start, MainManager.`map`.`ylimit` is set to -150.0 meaning if the player's y position ever gets lower than this, they will be respawned. This can override the map configuration
    - Any entity other than the player colliding with the hazard will be returned, but any NPCControl will have their `freezecooldown` set to 0.0 which will unfreeze them
    - The HazardAction animation when the player collides with the hazard is player y position being set to `holdspace` followed by the `Falling` sound (`SandFall` instead if the map is `DesertSandPitArea`)
- `WalkableSpike`: Spikey floor that can be walked on even if the player is shielded:
    - On Update, if MainManager.`player`.`shield` is true, the BoxCollider's center is set to offscreen at (2222.0, -99999.0, 88282.0) and if it's false, it's set to its original value from Start
    - The HazardAction animation when the player collides with the hazard (when they weren't shielded) is the `Damage0` sound followed by them spinning for 0.75 seconds
    - If returning an item entity, this is the only hazard type that won't lead to the destruction of the entity's GameObject
- `Honey`: A body of honey
    - On Start, MainManager.`map`.`lastwater` is set to the attached GameObject which can control how [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)'s sprout renders
    - Icicle platforms are pooled, but this hazard doesn't allow them to exist
    - Any entity colliding with the hazard will instantiate a `Prefabs/Objects/HoneySplash` prefab at the collider's GameObject and be destroyed in 0.75 seconds
    - The HazardAction animation when the player collides with the hazard is player y position being set to `holdspace` 
- `TempFire`: A large flame where it is assumed an Animator is attached in a child GameObject
    - This hazard starts inactive and it will every `speed` amount of frames toggle its activity between on or off (managed in Update when `alwaysactive` is true or the game is unpaused). If the hazard becomes active, the child Animator will have the `Start` clip played and the `Stop` clip when inactive. The activity also changes the BoxCollider's center: it is set to its initial value from Start when active and set to offscreen at (2222.0, -99999.0, 88282.0) when inactive
    - The HazardAction animation when the player collides with the hazard is the `Damage0` sound followed by them spinning for 0.75 seconds
    - If the entity that collides with the hazard follows someone, they will not be returned

## Other notes
Here are some other important details about the logic of the component:

- If the attached GameObject has the `WaterFloat` tag, MainManager.`map`.`waterfloat` is set to the attached GameObject which allows entities to move over it if they are allowed
- On HazardAction when the player collides with the hazard, there is a failsafe where if a player respawn happens 4 times in succession each within less than 60.0 frames apart, their position will be set to MainManager.`player`.`lastloadzone` + MainManager.`MainCamera` forward vector * 0.1 * the `playerdata` index (z fighting countermeasure). Normally, the base position used is MainManager.`player`.`lastpos`
- On HazardAction when the player collides with the hazard, StopForceMove is called without targetstate or smooth on the player's entity
- HazardAction contains some additional details when the player collides with the hazard:
    - The hazard animation includes a MainManager.[PlayTransition](../../General%20systems/Transition%20system.md)(0, 0, 0.2, Color.black) call at the start and a MainManager.PlayTransition(1, 0, 0.2, Color.black) call at the end, but these and most of the yield times will be skipped when in a `pause`, `minipause`, `inevent` or when the [message](../../SetText/Notable%20states.md#message) lock is held
    - All active NPCControl [Enemy](../../Entities/NPCControl/Enemy.md) that is within 4.0 units of the player gets their `freezecooldown` set to 0.0 which unfreezes them if they were
    - All Hornable of type `IceCube` within 4.0 units of the player has [ShatterDroppletIce](../../Entities/NPCControl/ObjectTypes/Dropplet.md#shatterdroppletice) called on their `parent` NPCControl
    - All ScrewPlatform's `camchange` are reset to false and if ScrewPlatform.`camischanging`, MainManager.[ResetCamera](../../General%20systems/Camera%20system.md#resetcamera) is called before resetting the value back to false
- The way icicle platforms work is if one is created when there's already `platformlimit` active one, the oldest one (tracked in an internal 2 elements array) will be destroyed to leave room for the new one. They always appear at the GameObject's position who collided with the hazard. The original GameObject gets destroyed when this happens
- The MainManager.`player`.`beemerang` won't cause a reaction when colliding with the Hazard
- Some specific entities won't react to colliding with the hazard. Here are the list that will simply stay on the hazard:
    - Any entity with `fixedentity` true
    - Any entity with `ignorewater` true
    - Any NPCControl [Enemy](../../Entities/NPCControl/Enemy.md) whose entity.`height` and entity.`minheight` are above 0.05 except if their entity.`killonfall` is true where it will react on top of their entity.`iskill` and entity.`dead` being set to true. NOTE: All NPCControl enemies will have their entity.`emoticoncooldown` will still be reset to 0.0 when colliding, but no further reaction will happen if it doesn't fall within the entity.`killonfall` case
    - NPCControl object of type [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md)
    - NPCControl object of type [Item](../../Entities/NPCControl/ObjectTypes/Item.md)
    - NPCControl object of type [Beemerang](../../Entities/NPCControl/ObjectTypes/Beemerang.md) (wouldn't cause a reaction anyway as mentioned above)
    - NPCControl object of type [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md)
    - NPCControl object of type [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md)
    - NPCControl object of type [PathPlatform](../../Entities/NPCControl/ObjectTypes/PathPlatform.md)
    - NPCControl object of type [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)
    - NPCControl object of type [RollingRock](../../Entities/NPCControl/ObjectTypes/RollingRock.md)
