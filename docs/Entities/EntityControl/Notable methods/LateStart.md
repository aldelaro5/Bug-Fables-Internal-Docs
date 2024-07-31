# LateStart
LateStart is a method that is only called on the first [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md) cycle of the entity. This is tracked using the `setup` field which is only set to true at the end of this method.

* Check the tag of the entity
    * For `Follower`, `PFollower`, `Player`, `NPC` and `Enemy`, `movesmoke` is initialised unless it is an item. Either way, the `anim`'s cullingMode is set to CullCompletely which disabled animations when not visible
    * For `Follower` and `PFollower`, [CreateDetector](../EntityControl%20Methods.md#createdetector) is called with a size of (0.0, 0.7, 0.3) and a center of (0.0, 0.5, 0.65). Collisions between `detect` and the `following` entity are ignored before setting `isfollower` to true.
    * For `Player`, `isplayer` is set to true and CreateShield called if it is the battle entity version of the player which initializes `bubbleshield` if it hasn't been already
* If it's an [item entity](../Item%20entity.md), set `overrideanim` and `overrridejump` as well as `oldstate` to -1 (`None`) and call `CreateFeet` which will initialise `feet` and set this entity as its parent
* Set `originalmap` to the current map
* If there is an `npcdata`
    * Set `npcdata`'s `currentdialogueindex` to NPCControl.GetDialogueIndex which is a very cut down version of [GetDialogue](../../NPCControl/Notable%20methods/GetDialogue.md#getdialogue) that only supports `npcdata.overridediag` and will only return the first applicable dialogue line id or -1 if none applied
    * If the map entity is a [PushRock](../../NPCControl/ObjectTypes/PushRock.md) with `internalcollider` present, ensure `feet` is initialised and call `CreateFeet` if not while also ignoring all collisions between `feet` and each `internalcollider`
* If by this point, `feet` hasn't been initialised and the entity is a `Follower`, `PFollower`, `isplayer`, `NPC`, `Enemy` or an `npcdata` of type [PushRock](../../NPCControl/ObjectTypes/PushRock.md) or [Item](../../NPCControl/ObjectTypes/Item.md), `CreateFeet` is called
* Set the `transform`'s position to `startpos` if it was assigned before or set `startpos` to the `transform`'s position if it wasn't
* Set `startbf`, `startbs` and `truescale` to `bobrange`, `bobspeed` and `startscale` respectively
* Call [UpdateMoveSmoke](../Update%20process/UpdateMoveSmoke.md)
* Set `oldstate` to -1 (None)
* Set `setup` to true which will prevent the game from calling this method again for this entity
