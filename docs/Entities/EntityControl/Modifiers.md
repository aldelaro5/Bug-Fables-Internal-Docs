# Modifiers
Modifiers are special piece of text that when present in the name of an [Entity](../Entity.md), it will alter its behavior. While most will apply when the modifier is contained in the name, some expect them to prefix the actual name so it is recommended to prefix them rather than putting them in the middle. It is possible to combine them. All of them are case sensitive.

Here are all the modifiers that are known:

|Modifier|Description|
|--------:|-----------|
|Holo|Sets `hologram` to true on [Start](Start.md) which makes the entity rendered as a hologram|
|TIME|Sets `extratimer` to true on [Start](Start.md) which doubles the time before the forcemove failsafe triggers|
|COT|Sets `hologram` and `cotunknown` to true which makes the entity rendered as a hologram in silhouette. This also calls RefreshCOT in 0.1 seconds which will set the materials of the extras accordingly to cot while setting `refreshedcotu` to true|
|ShwKEY|Sets `showitem` to true on [Start](Start.md) if the entity has an `npcdata` which renders a key item on top of the entity (defined in npcdata's data)|
|ICE|Sets the npcdata's `extrafreeze` to true on [Start](Start.md) if the entity has an npcdata TODO: Involves [NPCControl](../NPCControl/NPCControl.md)|
|Fixed|Sets `fixedentity` to true on [Start](Start.md) and calls SetFixed in 0.1 seconds which will completely lockup the rigid (including disabling gravity, making it kinematic and resetting the velocity to zero), disable the ccol and force the position to be startpos|
|FxdCol|A lighter version of Fixed which does everything it does except without disabling the ccol so it stays enabled|
|ALW|Sets `alwaysactive` to true on [Start](Start.md) which forces `incamera` to true and the entity remains active even out of the camera's range|
|ALF|Sets `alwaysflip` to true on [Start](Start.md) which calls [UpdateFlip](Update%20process/UpdateFlip.md) every LateUpdate|
|PAU|Sets `activeonpause` to true on [Start](Start.md) which allows the entity to receive updates and late updates on pause, minipause, [message](../../SetText/Notable%20states.md#message) lock and being dead|
|HIDE|Sets `hideinside` to true on [Start](Start.md) TODO: Involves [NPCControl](../NPCControl/NPCControl.md)|
|ROT|LateAngle is called on [CreateEntities](CreateEntities.md) in 0.25 seconds to set the angle instead of setting it direction. It also sets `lockrotater` to true on [Start](Start.md) which locks the y angle of the rotater.|
|ShwEm|Sets `alwaysemoticon` to true on [Start](Start.md) TODO: mostly involves [NPCControl](../NPCControl/NPCControl.md)|
|COG|Sets `startpos` to the point a raycast will hit from the transform heading down on [Start](Start.md)|
|NGS|Sets `onground` to false which forces the GroundDetector to report the entity being in the air on [Start](Start.md)|
|NGF|If this has an `npcdata` of type [Enemy](../NPCControl/Enemy.md) and it is cleared for being enabled on its Start, the [GravityFix](../NPCControl/GravityFix.md) coroutine is called at the end of its Start (this does nothing otherwise)|
|ITHD|Calls npcdata's SetHitInteract with HornDash on [Start](Start.md) if the entity has an npcdata|
|ITAH|Calls npcdata's SetHitInteract with AnyHorn on [Start](Start.md) if the entity has an npcdata|
|NEAR|When CheckNear is called, the entity is destroyed if it is further away than 30.0 units from the player|
|NDTCT|[HasHiddenItem](../NPCControl/Notable%20methods/HasHiddenItem.md) always returns false making this entity never trigger the Detector [medal](../../Enums%20and%20IDs/Medal.md) if the player has it equipped|
|DDIST|[HasHiddenItem](../NPCControl/Notable%20methods/HasHiddenItem.md) always returns false if the distance between this entity and the player is 20.0 or above making this entity never trigger the Detector [medal](../../Enums%20and%20IDs/Medal.md) if the player has it equipped|
|TOD|Allows a [Switch](../NPCControl/ObjectTypes/Switch.md) NPCControl object to be toggleable if its `data[0]` is 1 and `data[1]` is negative instead of only being able to be turned on|