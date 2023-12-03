# LateUpdate
This is the LateUpdate of EntityControl. This is the main update loop of the component and it is done on late due to the guarantee that coroutines have all ran by then so anything that could have edited the fields like the [animstate](../../Animations/animstate.md) have done so already.

* Every 3 frames
    * If paused, ensure the `transform`'s position is `pausepos` unless `activeonpause` is true, we are in an event, `iskill` is true or `dead` is true
    * Calls `UpdateCamPos` which updates `incamera`  with whether the entity is in the main camera range (ranging from (-0.6, -1.25) to (1.6, 2.25) in viewport coordinate while also being higher than -1 on the z axis) or true if `alwaysactive` is true, `camdistance` to the distance of the entity to the main camera and `campos` being the viewport position of the entity using the main camera
* Ensure `latetrans`'s position is set to `latepos` when applicable
* If we aren't paused and there's a `forcemove` in progress, handle the failsafe logic
    * Decrease the `forcetimer` by framestep if `playerentity` is true or that either `alwaysactive` is true or the entity is `incamera` and is within 30.0 units on the z axis unless `disabletimer` is true which will bypass this decrease for any entity except `playerentity`
    * If the `forcetimer` expired, the entity isn't `digging` and the player isn't free or it's not a `playerentity` then the failsafe will take effect by setting the `transform`'s position to `forcetarget`. Smokes will appear at the entity position before the failsafe and at `forcetarget` after the failsafe if it isn't `shrink` and its scale's magnitude is higher than 0.2.
* Handle the numb animation update logic if it is a battle entity while we are in a battle, the entity isn't `dead` or `iskill`, the battledata of the entity at `battleid` indicates `isnumb` and the battle hasn't `cancelupdate` by calling Numb. This method handles setting the [animstate](../../Animations/animstate.md) accordingly as well as randomly displacing the entity every once in a while with a yellow color
* Handles the `bubbleshield` position update if it is present
    * If the scale's magnitude is higher than 0.15, the default is to set the local position of the `bubbleshield` to (0.0, `height` + 1.25, 0.0), but `overrideshieldpos` being true can change this to be (`overrideshieldpos`.x, `height` + `overrideshieldpos`.y, `overrideshieldpos`.z)
    * Otherwise, the local position is set offscreen to (0.0, -999.0, 0.0)
* If the `hpbar` is present and active, ensure its local position is at (0.0, -0.5 + camoffset.y - battlecampos.y, 0.0) unless it's a `KeyR` which changes the x to 0.5 or a `Pitcher` which changes the x to -0.5
* If the `sprite` is present and the `animid` isn't `None` (-1), [RefreshTrail](../RefreshTrail.md) is called which will update the `traildata` according to `trail`
* Ensure the `transform` is childed to the map if it's a `tempfollower` and the `transform` doesn't have a parent already
* If `setup` is false, call [LateStart](../../Notable%20methods/LateStart.md)
* As long as the game isn't paused, call [Follow](../../Notable%20methods/Follow.md) which updates the following procedure
* If `alwaysflip` is true, call [UpdateFlip](../UpdateFlip.md)
* If the game is unpaused and `incamera`, the `incamera` update will happen if we are in an event OR that the `campos` z is less than 25.0 OR less than 30.0 only if the `npcdata`'s `startlife` is smaller than 50.0 / not present
    * Assign `truescale` to `startscale`
    * If the `icecube` is present, set its scale by a lerp of its current scale to `freezesize` with a factor of framestep * 0.25 and set its position by `freezeoffset` + Vector3.up * `height` or by (`freezeoffset`.x + Random.Range(-0.1, 0.1), `freezeoffset`.y + `height`, `freezeoffset`.z + Random.Range(-0.1, 0.1)) if `shakeice`
    * If `springcooldown` is active while the `rigid`'s velocity is descending on the y axis or `onground` is true, deactivate the cooldown
    * General update methods are called under conditions:
        * [UpdateGround](../UpdateGround.md), [UpdateAirAnim](../UpdateAirAnim.md) and [UpdateVelocity](../UpdateVelocity.md) are called every 2 frames except for `battle` entity or the `npcdata` is not NPC / not present where they will be called regardless of the current frame
        * [UpdateGeneralAnim](../UpdateGeneralAnim.md) is called every 2 frames
        * [UpdateEmoticon](../UpdateEmoticon.md) is called when in an event or it is a `battle` entity or every 2 frames
        * [UpdateMoveSmoke](../UpdateMoveSmoke.md) is called every 3 frames
        * [RefreshShadow](../RefreshShadow.md) is called when `isfollower` is true and the player is flying OR that there is no `npcdata` OR that it has one but it's also an Enemy, a SemiNPC for every 10 frames or it's an NPC/Object every 2 frames in addition that the `insideid` matches the current one
        * [UpdateStatusIcons](../UpdateStatusIcons.md) and [UpdateSound](../UpdateSound.md) are always called
        * [UpdateFlip](../UpdateFlip.md) is called either when we are not in a battle or the entity itself is a `battle` entity (if `alwaysflip` was true, this is means this can be called a second time)
* Otherwise, the inactive update will happen
    * The entity is muted when applicable when `soundonpause` is true OR that it's not a battle battle and it either has no `npcdata` or it has one, but it's a `MusticRange` Object, otherwise, [UpdateSound](../UpdateSound.md) is called instead
    * Reset the `emoticon` to none and the `emoticoncooldown` to 0.0 when applicable
    * Deactivate the `movesmoke` if it is present and enabled
* This is the general update handling when unpaused that applies to all entities
    * Every 3 frames, `UpdateRotater` is called which will make sure the `rotater`'s y angle matches the main camera one unless `lockrotater` is true. If we were not in an event [UpdateCollider](../UpdateCollider.md) is called before it, but because this one only process every 2 frames, it will actually only process every 6 frames effectively.
    * Rendering updates happen if the entity is `incamera`, it is a `battle` entity or that we are in an event
        * [UpdateHeight](../UpdateHeight.md) is called when it's a battle entity or we are in an event or `alwaysactive` is true or it has no `npcdata` or it has one, but it's not an NPC or it is an NPC, but the `startlife` is below 50.0
        * [AnimSpecificQuirks](../../Animations/AnimSpecific.md#animspecificquirks) and [UpdateSprite](../UpdateSprite.md) are called
        * `flyinganim` is set to true if the `height` is higher than 0.1 unless `overridefly` is true which forces it to false
    * If we aren't in an event, then `offgroundframes` is increased by framestep if it hasn't reached 1000.0 and that `onground` is false, otherwise, it is set to 0.0. If we are in an event, that it is decreased by framestep if `onground` is true and it hasn't gone below 0.0
    * If the `jumpcooldown` has expired and `stopspinonground` is true, it is set to false on top of zeroing out `spin`
* `lastpos` is set to the `transform`'s position
* `oldground` is set to `onground`
