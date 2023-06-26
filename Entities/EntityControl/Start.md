# Start

EntityControl defines a Start which performs several important steps. This will be run at the start of the next frame after the creation occurred, but also after being enabled if the entity started disabled:

* Ensure `transofrm` and `spritetransform` are set correctly
* Ensure `icecubeprefab` is set to the `Prefabs/Objects/icecube` object from the root of the asset tree and set the object's layer to 13 (`NoDigGround`)
* Add a RigidBody component assigned to `rigid` with the constraints set to freeze rotations
* Add an Animator component assigned to `anim`
* Initialise `digpart` to an array of 2 new game objects
* Assign the `rotater` created earlier by getting the first child of the root
* Apply the `Holo`, `TIME` and `COT` [Modifiers](Modifiers.md) logic as well as `ShwKEY` and `ICE` if this entity has an `npcdata`'s [NPCControl](../NPCControl/NPCControl.md)
* Set the sprite's material to the game's sprite material or the hologram one depending on the value of `hologram`
* Set the sprite's object's layer to 14 (`Sprite`)
* Set the sprite to receive shadows
* Ensure there's a CapsuleCollider assigned to `ccol` and set it's material to the game's default physics material
* Set `initialcenter` to the `ccol`'s center and `initialcolliderdata` to (`ccol`'s height, `ccol`'s radius)
* If the current map is MetalLake, set `overridemovesmoke` to true. If the map's `icemap` is true, then `inice` is set to true
* Ensure `startpos` is set to the transform's position
* Set `spawnpoint` to the transform's position
* Unless `noemoticon` is set to true, setup the `emoticon` object which is a new object with an Animator named `Emoticon` childed to the `rotater`, a local position of `emoticonoffset`, and a layer of 15 (`3DUI`). Also set `emoticonsprite` to a new SpriteRenderer added to `emoticon` and set its material to Unity's default sprite material
* Ensure an AudioSource is added and assigned to `sound` without playOnAwake
* If `startvelocity` is set, set the `rigid`'s velocity to it
* Call [CheckSpecialID](Notable%20methods/CheckSpecialID.md) which loads the \[\[TextAsset Data/Entity data#`EntityValues` data|Entity data\]\]
* Set the entity's bleep. For more information on the bleep system, check [Bleep](../../SetText/Commands/Individual%20commands/Bleep.md). The bleep id and pitch are expected to be set beforehand in `dialoguebleepid` and `bleeppitch` respectively.
* If `hasshadow` is true, ensure a an object named `shadow` with a SpriteRenderer is created and assigned to the field of the same name and childed to the entity's transform at (0.0, -999.0, 0.0). The transform of the new object is assigned to `shadowtransform`. The sprite of the shadow is set to the game's shadow sprite with a color of pure white with 40% transparency. The angles of the object are set to (90.0, 0.0, 0.0) and the material's renderQueue is set to 2900
* If the entity is an item, assign `itemstate` to the `animstate`
* Apply the following [Modifiers](Modifiers.md): `Fixed`, `FxdCol`, `ALW`, `ALF`, `PAU`, `HIDE`, `ROT`, `ShwEm`, `COG`, `NGS` when applicable
* If the entity has an `npcdata`, also apply the [Modifiers](Modifiers.md) `ITHD` and `ITAH` when applicable
* If we are in a battle and this is the player's entity, ensure `bubbleshield` is created. This will instantiate the `Prefabs/Objects/BubbleShield` prefab from the root of the asset tree and add a DialogueAnim component to it which grants it the ability to hide using grow/shrink animations. The object is childed to the `rotater` and it starts shrunk with the shrink speed set to 0.075 and the targetscale when revealed to (1.0, 3.15, 1.0). The object is initially set to have a scale of 0 and a local position of (0.0, 1.25, 0.0). The Renderer of the new object has its color set to full white with 55% transparency and a renderQueue of 2505
* `lastpos` is set to the `startpos` assigned earlier
* `initialheight` is set to `height`
* Unless `overrideminheight` is true, the `height` is set to `minheight` if the height was smaller than the minimum
* Set the sprite's tranform's angles accordingly to whether or not `flip` is set. If it is, it's rotated 180 degrees on the y axis and if it's not, 0 degrees
* Set `playerentity` to true only if the tag of the entity is `Player` or `PFollower`.
