# EntityControl Methods

There are all the methods in [EntityControl Creation](EntityControl%20Creation.md).

## Unity Events

This component implements `Start`, `OnEable` `Update`, `LateUpdate`, `FixedUpdate`, `OnTriggerStay` and `OnTriggerExit`. You can lean more about them in [Start](Start.md), [Update](Update%20process/Unity%20events/Update.md), [LateUpdate](Update%20process/Unity%20events/LateUpdate.md), [FixedUpdate](Update%20process/Unity%20events/FixedUpdate.md), [OnTriggerStay](Update%20process/Unity%20events/OnTriggerStay.md) and [OnTriggerExit](Update%20process/Unity%20events/OnTriggerExit.md). Notably, `Start`, `OnEnable` and the 3 updates ones are an integral part of the startup and update process of the entity.

## Creation

These methods are the gateway to initialise entities. As such, they are static and returns an EntityControl except for items which gives an [NPCControl](../NPCControl/NPCControl.md). No matter the context to create the entity, it has to go through one of these

### CreateNewEntity

These overloads are fully documented at [EntityControl Creation > Creating an entity](EntityControl%20Creation.md)

````cs
public static EntityControl CreateNewEntity(string name)
````

````cs
public static EntityControl CreateNewEntity(string name, int anim_id, Vector3 position)
````

````cs
public static EntityControl CreateNewEntity(string name, int anim_id, Vector3 position, EntityControl follow)
````

### CreateItem

Items uses a different method which is more a helper to create an item as an NPC. TODO: this will be documented at a later time since most of this involves [NPCControl](../NPCControl/NPCControl.md)

````cs
public static NPCControl CreateItem(Vector3 startpos, int itemtype, int itemid, Vector3 direction, int timer)
````

## Lifecycle methods

These methods are called by Unity events during key moments in the entity lifecycle. Some of them can be called outside of them, but being part of the lifecycle is their main purpose.

### LateStart

This method uses the `setup` field as a fire and forget which means it will only be called once during the first [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) and then never called again. More details at [LateStart](Notable%20methods/LateStart.md).

````cs
private void LateStart()
````

### Update methods

These methods were eventually called by [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) and they each manage different parts of the update process. Since they are essential, dedicated pages are made for them.

[UpdateAnimSpecific](Animations/AnimSpecific.md#updateanimspecific)

````cs
public void UpdateAnimSpecific()
````

[AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks)

````cs
private void AnimSpecificQuirks()
````

[UpdateItem](Update%20process/UpdateItem.md)

````cs
public void UpdateItem()
````

[CheckSpecialID](Notable%20methods/CheckSpecialID.md)

````cs
public void CheckSpecialID()
````

[RefreshTrail](Update%20process/RefreshTrail.md)

````cs
private void RefreshTrail()
````

[UpdateFlip](Update%20process/UpdateFlip.md)

````cs
private void UpdateFlip()
````

[UpdateStatusIcons](Update%20process/UpdateStatusIcons.md)

````cs
private void UpdateStatusIcons()
````

This method updates `incamera`  with whether the entity is in the main camera range (ranging from (-0.6, -1.25) to (1.6, 2.25) in viewport coordinate while also being higher than -1 on the z axis) or true if `alwaysactive` is true. It will also update `camdistance` to the distance of the entity to the main camera and `campos` to the viewport position of the entity using the main camera

````cs
private void UpdateCamPos()
````

[UpdateSound](Update%20process/UpdateSound.md)

````cs
private void UpdateSound()
````

[UpdateSprite](Update%20process/UpdateSprite.md)

````cs
private void UpdateSprite()
````

[UpdateHeight](Update%20process/UpdateHeight.md)

````cs
private void UpdateHeight()
````

[UpdateVelocity](Update%20process/UpdateVelocity.md)

````cs
private void UpdateVelocity()
````

This will ensure the `rotater`'s y angle matches the main camera one unless `lockrotater` is true

````cs
private void UpdateRotater()
````

[UpdateGeneralAnim](Update%20process/UpdateGeneralAnim.md)

````cs
private void UpdateGeneralAnim()
````

[UpdateAirAnim](Update%20process/UpdateAirAnim.md)

````cs
private void UpdateAirAnim()
````

[UpdateCollider](Update%20process/UpdateCollider.md)

````cs
private void UpdateCollider()
````

[UpdateEmoticon](Update%20process/UpdateEmoticon.md)

````cs
private void UpdateEmoticon()
````

[UpdateMoveSmoke](Update%20process/UpdateMoveSmoke.md)

````cs
private void UpdateMoveSmoke()
````

[RefreshShadow](Update%20process/RefreshShadow.md)

````cs
public void RefreshShadow()
````

[UpdateGround](Update%20process/UpdateGround.md)

````cs
private void UpdateGround()
````

## Movement

These methods manages movement of the entity. Whether it is through `forcemove` or moving in general, they manage their movement taking into account animations and colliders.

### SetPosition

Sets the `transform`'s position, `startpos` and `lastpos` to `pos`.

````cs
public void SetPosition(Vector3 pos)
````

### DelayedPosition

Sets the entity's position to be `pos` after `time` seconds have passed.

````cs
public void DelayedPosition(Vector3 pos)
````

````cs
public void DelayedPosition(Vector3 pos, float time)
````

The default value of `time` is -1.0 (one frame).

Both of these overloads ends up starting a SetLatePos coroutine with the parameters sent:

````cs
private IEnumerator SetLatePos(Vector3 pos, float time)
````

The coroutine will yield for the amount of `time` in seconds (or a frame if it was 0 or below) before setting the `transform`'s position to `pos`. Unlike SetPosition, this does not set `startpos` and `lastpos`.

### Jump

Process a jump with velocity `h`

````cs
public void Jump()
````

````cs
public void Jump(float h)
````

The default value of `h` is `jumpheight` which itself defaults to 10.0

This will first call Unfix if it's not an `item` and we aren't in a battle. After, `offgroundframes` is set to 20.0, `jumpcooldown` to 30.0  and the `rigid`'s velocity to have `h` in the y, but leave x and z unchanged.

As a special case, if a `flowerbed` is present, the `FlowerJump` particle is played.

### ForceMove

Forces the entity to move to `target` after `frametime` amount of frames has passed maintaining the `movestate` [animstate](Animations/animstate.md) during the move and `stopstate` when it is completed

````cs
public void ForceMove(Vector3 target, float frametime, int movestate, int stopstate)
````

There's also an overload that changes the last 2 parameter into `changeanim` where either [animstate](Animations/animstate.md) is the current one unless true is passed to this parameter where `movestate` will be `walkstate` and `stopstate` will be `basestate`. Basically, this parameter controls if the animation should be left unchanged during the move or change to the defaults states.

````cs
public void ForceMove(Vector3 target, float frametime, bool changeanim)
````

Both of these overloads ends up calling the coroutine version storing the call into `forcemoving` (set to null on return):

````cs
private IEnumerator ForceMove(Vector3 target, float frametime, int[] an)
````

The way the force move works is the coroutine entirely manages the force move unlike the MoveTowards version which is managed via [FixedUpdate](Update%20process/Unity%20events/FixedUpdate.md) and fields.

The actual move is done by setting the `transform`'s position to a lerp between the position we started with to `target` with a factor of `a` / `frametime` where `a` is the amount of frames that elapsed since the start of the call. This lerp keeps being redone with a frame yield between iterations until `a` reached `frametime` + 1.0.

Basically, this is a much more aggressive version than MoveTowards because this will always succeed in moving the entity even if it might not physically make sense. This is best used if the game can guarantee that moving to the desired position will make sense.

### StopForceMove

Completely halts any ongoing ForceMove or MoveTowards process of force moving either by completely stopping the velocity in x and z or by decelerating at half the current one if `smooth` is false. The [animstate](Animations/animstate.md) is set to `targetstate` if above -1 once this is done and the entity is at rest.

````cs
public void StopForceMove()
````

````cs
public void StopForceMove(int targetstate, bool smooth)
````

The default value of `targetstate` is -1 (no [animstate](Animations/animstate.md) changes) and the default value of `smooth` is false (complete velocity stops).

This will set `forcemove` to false and stop `forcemoving` if it was active before setting it to null. Then, either StopMoving is called sending `targetstate` in the case of `smooth` being false or if it's true, the `rigid`'s velocity is set to a lerp between the current one and the current y one with x and z zeroed out. This is done with a factor of 0.5 which will be the midpoint between the current x z velocity and a complete stop. From there, the [animstate](Animations/animstate.md) gets set to `targetstate` if above -1, `overrideanim` is false and the velocity's magnitude ended up \<= 0.1.

Finally, If `looktowards` had a value, DelayedLook is called with it with a delay of 0.22 seconds before setting it to null

````cs
private IEnumerator DelayedLook(Vector3 target, float delay)
````

### Move

More details available at [Move](Notable%20methods/Move.md), this manages the velocity and angles during an instant of a movement respecting `walktype` during the move.

````cs
public void Move(Vector3 pos, float multiplier, int state)
````

### StopMoving

Stops the x and z velocity, stops any acceleration (`deltavelocity`) and set the [animstate](Animations/animstate.md) to `targetstate` if it is above -1 (otherwise, it is set to `basestate` if it was matching `walkstate`)

````cs
public void StopMoving(int targetstate)
````

### MoveTowards

Setup a `forcemove` through [FixedUpdate](Update%20process/Unity%20events/FixedUpdate.md) to go to `pos` with a speed multiplier of `multiplier` setting the [animstate](Animations/animstate.md) to `state` during the move and `stopstate` once it is completed where the y axis is ignored if `ignore_y` is true.

````cs
public void MoveTowards(Vector3 pos)
````

````cs
public void MoveTowards(Vector3 pos, float multiplier)
````

````cs
public void MoveTowards(Vector3 pos, float multiplier, int state, int stopstate)
````

````cs
public void MoveTowards(Vector3 pos, float multiplier, int state, int stopstate, bool ignore_y)
````

The defaults values are:

* `multiplier`: 1.0
* `state`: `walkstate`
* `stopstate`: `basestate`
* `ignore_y`: false

There is also an overload available that calls the same one above, but also sets `looktowards` to `lookaftermove`.

````cs
public void MoveTowards(Vector3 pos, float multiplier, int state, int stopstate, bool ignore_y, Vector3 lookaftermove)
````

Finally, 2 overloads are available to only pass `pos` with the separate components

````cs
public void MoveTowards(float x, float y, float z)
````

````cs
public void MoveTowards(float x, float z)
````

The default `y` is 0.0.

This will assign the force move related fields to their proper value and return. This is done because the actual move is managed completely by [FixedUpdate](Update%20process/Unity%20events/FixedUpdate.md) and the failsafe for it is handled in [LateUpdate](Update%20process/Unity%20events/LateUpdate.md). 

It is not to be confused with the ForceMove coroutine which completely handles the move itself. Because the update process handles this move, it's a less aggressive one and it allows the entity to call Jump if needed. This gives the guarantee that the entity will try to move to `pos` in the most logical way. If it takes too long, the failsafe will trigger during [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) and the entity will be instantly moved.

Here are the actual values that will be set to the force move fields:

* `forcemultiplier`: `multiplier`
* `forcetarget`: `pos` + the normalised direction towards `pos` * 0.4 which only slightly overshoots `pos`
* `forcestop`: `stopstate`
* `forceanim`: `state`
* `ignorey`: `ignore_y`
* `forcetimer`: It depends if it's the `playerentity`. If it is, the value is 375.0 if we're in an event or 250.0 if we're not. If it's not the `playerentity`, the value is 225.0 if it's a `battle` entity while a CheckDead is running or 500.0 otherwise.

Finally, `forcemove` is set to true which activates it.

## Flip and angles adjustments

These methods adjusts the angles of the `rotater` which can either be due to the need to look somewhere or to spin or simple `flip` logic.

### FaceTowards

Changes the `flip` and `backsprite` accordingly to the entity facing towards `other`. `backsprite` will not be changed if `noback` is true unless `forceback` is true.

````cs
public void FaceTowards(Vector3 other)
````

````cs
public void FaceTowards(Vector3 other, bool noback)
````

````cs
public void FaceTowards(Vector3 other, bool noback, bool forceback)
````

The default value for `noback` and `forceback` are false. However, `noback` is not honored on the second overload and false is sent instead. The third overload must be used to sent `noback` to true. This means the second overload is practically useless.

This will change `flip` by the value of the x viewpoint of the `transform` \< the x viewpoint of the `other`. Basically, true is if the entity is positioned strictly to the left of `other`, false otherwise.

For `backsprite`, the assignment happens regardless if `forceback` is true, but if it's not then all of the following condition needs to be met to assign a new value:

* `noback` is false
* `lockback` is false (this field is unused so this is always the case normally)
* The entity is not `talking`
* The [message](../../SetText/Global%20vars%20used/message.md) lock is released

As for the new value of `backsprite`, it's the value of the z viewpoint of `other` + -0.5 > the z viewpoint of the `transform`. Basically, it's true if after placing `other` 0.5 units towards the camera, it remains positioned more away from the camera than the entity, false otherwise.

### FaceTowards helper methods

These methods just calls FaceTowards with a specific `pos` as the only parameter.

Faces towards the player if it is present

````cs
public void FacePlayer()
````

Faces towards the current position + (1.0, 0.0, 0.0)

````cs
public void FaceAhead()
````

Faces towards the current position + (-1.0, 0.0, 0.0)

````cs
public void FaceBehind()
````

Faces towards the current position + (0.0, 0.0, 1.0)

````cs
public void FaceUp()
````

Faces towards the current position + (0.0, 0.0, -1.0)

````cs
public void FaceDown()
````

### FlipSpriteAngleAt

Set the `spritetransform` to look at `target` and set its angle to (0.0, current y angles, 0.0) + `offset`. This also sets `overrideflip` to true.

````cs
public void FlipSpriteAngleAt(Vector3 target)
````

````cs
public void FlipSpriteAngleAt(Vector3 target, Vector3 offset)
````

The default `offset` is zero.

### Flipping methods

These methods manages the `flip` field which controls which side the sprite is rendered.

Toggles `flip`

````cs
public void FlipSimple()
````

Returns a vector corresponding to the current `spritetransform` in x and z, but the y component is 180.0 if `flip` is true and 0.0 is it is false. This essentially is a way to know the actual angles of the entity considering its current `flip`.

````cs
public Vector3 FlipAngle()
````

This will call the other overload if `setangle` is false. If it is true, it will actually set the angles of `spritetransform` on top of returning them.

````cs
public Vector3 FlipAngle(bool setangle)
````

A helper to get the speed the flip should occur in [UpdateFlip](Update%20process/UpdateFlip.md). The logic is unless there is an `npcdata` with a `startlife` of less than 50.0, this just returns `flipspeed`. If this is the case however, FlipAngle(true) is called and this returns 1.0.

````cs
private float GetFlipSpeed()
````

### Spinning methods

These methods manages the `spin` value which is an angle vector to spin the entity each [UpdateFlip](Update%20process/UpdateFlip.md)

Set `spin` to zero which stops the entity from spinning

````cs
private void StopSpin()
````

Set `spin` to `s`, but invoke StopSpin in `time` seconds which means the spin is temporary until StopSpin gets called.

````cs
public void TempSpin(Vector3 s, float time)
````

Set `spin` to `spinamount`, then have its value decrease smoothly towards zero over the course of `frametime` frames before finally have its value set to zero upon completion.

````cs
public IEnumerator SlowSpinStop(Vector3 spinammount, float frametime)
````

Before all this happens, the `spritetransform` scale is set to `startscale`. If the `model` exist and its scale's magnitude is below 0.1, its scale gets set to Vector3.one.

The actual value of `spin` during the deceleration is a lerp from the `spinamount` to zero with a factor of 1.0 - frames left / `frametime` with a yield of one frame between each iteration.

## Sounds

Play a sound named `clipname` from the `Audio/Sounds` directory using the `sound` audio source if it can be heard at pitch `pitch` before calling [UpdateSound](Update%20process/UpdateSound.md) and setting `soundvolume` to `volume` (which only sets the volume for the NEXT clip, not this one).

````cs
public void PlaySound(string clipname)
````

````cs
public void PlaySound(string clipname, float volume)
````

````cs
public void PlaySound(string clipname, float volume, float pitch)
````

The default value of `volume` and `pitch` are 1.0 which is normal volume and pitch.

All the three overloads above ends up calling this one which just loads the clip before sending it to the `clip` parameter

````cs
public void PlaySound(AudioClip clip, float volume, float pitch)
````

The sound will only be played if all of the following conditions are met:

* The game settings doesn't have the SFX volume muted and if we are currently configuring it in the pause menu that the current setting isn't muted
* `campos`.z is less than 25.0 which means the entity must not be 25.0 or more units away from the camera
* The game's globalcooldown expired
* `campos` is in the camera's range
* Either this is the `beemerang`, there is no `npcdata` or there is one with a `startlife` of 15.0 or more

This is an alias of the first overload.

````cs
public void PlaySoundSimple(string clipname)
````

Returns 1.0 - the clamp of `camdistance` from 0.0 to `soundistance` / `soundidstance` where `soundistance` is normally always 25.0. The return value is between 0.0 and 1.0 where 0.0 means the distance is too far for a sound to be heard and 1.0 means the sound could play at full volume as the entity is at the camera.

````cs
public float GetSoundDistance()
````

## Animations

These methods handles [animstate](Animations/animstate.md) and `animid` [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) changes.

### SetAnim

More details at [SetAnim](Animations/SetAnim.md)

````cs
public void SetAnim(string args)
````

````cs
public void SetAnim(string args, bool force)
````

There is a helper method to call it with empty `args` and `force` to true

````cs
public void SetAnimForce()
````

### SetState

Set the [animstate](Animations/animstate.md) to `state`

````cs
public void SetState(int state)
````

### ReturnToIdle

Set the [animstate](Animations/animstate.md) to `basestate`

````cs
public void ReturnToIdle()
````

### ExtraAnimPlay

If `extraanims` is present and not empty, either play an animation named `arg` to all the `extraanims` or play a specific one for each by having `arg` contain all the animation clip names separated by `,`. If the later is done, `arg` split by `,` MUST contain at least the amount of `extraanims` or an exception will be thrown.

````cs
public void ExtraAnimPlay(string arg)
````

### SpecialAnimation

Plays a specific animation routine named `animation` case insensitive with the call stored in `specialanim` (turns to null on return). This only supports one named `levelup` that will set specific routines for the `Bee`, `Beetle` and `Moth` [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md). Before the animation plays, `overrideanim` is set to true and `overridejump` is set to true with the old value saved. Once the animation is done, a frame is yielded, `overrideanim` goes back to false and `overridejump` is restored to its old value.

````cs
public IEnumerator SpecialAnimation(string animation)
````

### Animator change

Sets the runtimeAnimatorController of `anim` depending on the [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) with special handling for PUSHROCK and Hard Mode equipped on HARDEST. More details is at [SetAnimator](Update%20process/UpdateSprite.md#setanimator)

````cs
public void SetAnimator()
````

Wraps SetAnimator by first ensuring `anim` is created if it wasn't and calls [UpdateAnimSpecific](Animations/AnimSpecific.md#updateanimspecific) after the call.

````cs
public void ForceAnimator()
````

### Animation event

These methods are never called directly, but rather by an animation event. 

This one only applies only to the `CordycepsAnt` [AnimID](../../Enums%20and%20IDs/AnimIDs.md) where for `chance` % change when called, it will play the animation `Idle0` or `Idle1` with a 50/50 chance.

````cs
private void RandomAnimationEvent(int chance)
````

This only applies to `Bee` [AnimID](../../Enums%20and%20IDs/AnimIDs.md) and the `BattleIdle` [animstate](Animations/animstate.md). It will play the ParticleSystem on `animspecific` at index `index` if either are present after setting the angles of the `animspecific` object according to `flip`

````cs
public void PlayAnimSpecific(int index)
````

## Rigid and ccol management

These methods manages physics with the `rigid` and `ccol` fields.

### EnableCol

Enables the `ccol`

````cs
public void EnableCol()
````

### SetFixed

Apply most of the `Fixed` [Modifiers](Modifiers.md) after `fixedentity` is set to true on [Start](Start.md)

````cs
private void SetFixed()
````

### SetFixedCollider

Apply most of the `FxdCol` [Modifiers](Modifiers.md) after `fixedentity` is set to true on [Start](Start.md)

````cs
private void SetFixedCollider()
````

### LockRigid

Either locks the `rigid` when `value` is true by making it kinematic without gravity or unlock it when `value` is false by enabling its gravity without kinematic. This also zeros out the velocity if `resetvelocity` is true for either operations.

````cs
public void LockRigid(bool value)
````

````cs
public void LockRigid(bool value, bool resetvelocity)
````

The default value for `resetvelocity` is true.

### LateVelocity

Yield until `rigid` isn't null anymore and then set its velocity to `ammount`

````cs
public IEnumerator LateVelocity(Vector3 ammount)
````

### Unfix

If the proper conditions are met or `force` is true, SetFixed and SetFixedCollider are undone by calling LockRigid(false, false), enabling the `ccol`, setting the  `rigid` constraint to `FreezeRotation` and setting `fixedentity` to false.

````cs
public void Unfix()
````

````cs
public void Unfix(bool force)
````

The default value of `force` is false.

If `force` is false, the method only act if there is no `npcdata` and if there is one, it must not be a type Object or SemiNPC and its `interacttype` must not be CaravanBadge or Shop.

## Detector

These methods involves the `detect` and `feet` object which are ways to detect walls and grounds around the entity.

### CreateDetector

Creates and add `detect`, a RayDetector on a trigger box collider with size `size` and center `center` that detects walls and other entities. The result is maintained in `hitwall`.

````cs
public void CreateDetector()
````

````cs
public void CreateDetector(Vector3 size, Vector3 center)
````

The default `size` is (0.8, 0.7, 0.15) and the default `center` is (0.0, 0.5, 0.5)

The detector only detects collisions with objects in the layers Ground, NoDigGround and Entity. Once a collision is detected, `hitwall` turns to true until the collision has been exited when it goes back to false.

There is one exception to this rule. If the entity is the player, the [animstate](Animations/animstate.md) isn't 116 and there is a raycast hit from the detector forward with a max distance of 1.0 unit (using the same layers as filter), then the normal of the hitpoint must not have 95% or more of its direction be in any of the 3 axises. If this isn't the case, `hitwall` is set to false instead.

It should be noted that because of the detector being slightly next to the `ccol` under normal gameplay, it is possible to miss detection of thin wall colliders due to the detector rendering past the wall.

### CreateFeet

Create the `feet`, a GroundDetector with an octogon shaped trigger collider loaded from the `Prefabs/GroundDetector` prefab from the root of the asset tree that will detect the presence of grounds and maintain the result in `onground`.

````cs
public void CreateFeet()
````

The collider is configured to be at a local position of zero and a 0.1 vertical scale (the x and z ones are each set to `ccol`.radius * 2 - 0.25). This makes the collider very thin and positioned under the entity.

The collider detects any objects in layers Ground and NoDigGround. A collision causes `onground` to turn to true until its exit which makes it revert back to `false`. The detector also manages the ability to dig depending on which of the 2 layers collided. Additionally, the detector can handle special cases where the collider has the tag `PushPlatform`, `Platform` and `PlatformNoClock`. Finally, it also handles the smoke and sound effects being played when a PushRock lands.

### HasGroundAhead

Perform a Raycast downward starting from `point` with max distance `checkdistance` with layers Ground and NoDigGround and return whether there was a hit

````cs
public bool HasGroundAhead()
````

````cs
public bool HasGroundAhead(Vector3 point, float checkdistance)
````

The default value of `point` is `transform`.position + `detect`.transform.forward.normalized * 2.0 + Vector3.up / 2.0 and the default value of `checkdistance` is 5.0.

This overload will additionally call CreateDetector if `detect` doesn't exist and have `detect` look at `target` before calling the parameterless overload.

````cs
public bool HasGroundAhead(Vector3 target)
````

### DetectDirection

Call CreateDetector if `detect` doesn't exist, make `detect` look at `targetp` and zero out the x and z angles.

````cs
public void DetectDirection(Vector3 targetp)
````

### ForceHitWall

Set `hitwall` to true which forces `detect` to report a hit.

````cs
public void ForceHitWall()
````

### DetectIgnoreSphere

Have `detect` ignore collisions every laoded [NPCControl](../NPCControl/NPCControl.md)'s `scol`.

````cs
public void DetectIgnoreSphere(bool ignore)
````

## Structural additions

These methods manages diverse object hierarchy extensions.

Create `bubbleshield` if it didn't exist as an instance of `Prefabs/Objects/BubbleShield` from the root of the asset tree with a DialogueAnim that starts out shrunk with a shrink speed of 0.075 and a targetscale of (1.8, 3.15, 1.0).

````cs
public void CreateShield()
````

The starting scale is zero with a local position of (0.0, 1.25, 0.0), The material's color is set to pure white with 55% alpha and a renderQueue of 2505.

Create `line` which is a LineRenderer starting from `start` to `end` with width `width` on both ends with a color of `color` on both ends and a parent of `parent` (the object is not childed to anyone if `parent` is null).

````cs
public void CreateLine(Vector3 start, Vector3 end, float width, Color color, Transform parent)
````

The material of `line` is set to the spritemat

Returns the element of `extra` at index `id` if `anim` is false or the element of `extraasnims` at index `id` if `anim` is true

````cs
public GameObject GetExtras(int id, bool anim)
````

Create the `shadow` object which is a SpriteRenderer and initialises `shadowtransform` to its transform. The object will initially be rendered offscreen with sprite set to shadowsprite with a color of pure white with 40% alpha and a renderQueue of 2900.

````cs
public void CreateShadow()
````

Create `model` from an instance of `path` from the root of the ressources tree at local position `offset`. This sets `anim` to the `model`'s animator if it is present and it also handles `hologram` and `cotunknown` by changing the materials and colors of all the children SpriteRenderer, SkinnedMeshRenderer and MeshRenderer recursively.

````cs
public void AddModel(string path, Vector3 offset)
````

Creates the `hpbar` and `defstat` which are UI objects positioned underneath the entity that reports HP and Defense numbers.

````cs
public void CreateHPBar()
````

## Colliders ignoring

These methods allows to ignore collisions with entities or other colliders.

Have every collider under `a` recursive ignore every collider under `b` recursive if `ignore` is true or unignore them if it is false.

````cs
public static void IgnoreColliders(EntityControl a, EntityControl b, bool ignore)
````

Have every collider under `a` recursive ignore `b` if `ignore` is true or unignore if it is false.

````cs
public static void IgnoreColliders(EntityControl a, Collider b, bool ignore)
````

Have `ccol` and `detect` ignore collisions with `c` immediately until `sectime` seconds has passed at which point, all collisions that were ignored are unignored.

````cs
public IEnumerator TempIgnoreColision(Collider c, float sectime)
````

## Emoticon

These methods manages `emoticon` which is a UI object that renders on top of the entity to indicate possible interactions.

Set `emoticonid` to `emote` - 1 and `emoticoncooldown` to `time` before calling [UpdateEmoticon](Update%20process/UpdateEmoticon.md) which will show `emote` for `time` frames.

````cs
public void Emoticon(MainManager.Emoticons emote)
````

````cs
public void Emoticon(MainManager.Emoticons emote, int time)
````

````cs
public void Emoticon(int type, int time)
````

The first 2 overloads calls the third one which does all the work.

## Overrides

These methods manages the overrides fields which are fields to bypass normal behaviors in many places notably during the update cycle.

Set `overrideanim` to `animation`, `overrridejump` to `jumpanimation`, `overrideflip` to `flipbehavior`, `overridefly` to `flyanimation`, `overrideonlyflip` to `onlyflip` and `overrideanimspeed` to `animationspeed`.

````cs
public void SetOverrides(bool animation, bool jumpanimation, bool flipbehavior, bool onlyflip, bool flyanimation, bool animationspeed)
````

Calls SetOverrides with every parameter being false

````cs
public void ResetOverrides()
````

Set `overreidejump` to true immediately before yielding for 0.5 seconds, then yield frames until `onground` is false, then yield 0.5 seconds before finally setting `overrridejump` back to false.

````cs
public IEnumerator OverrideJumpTemp()
````

Set `overrideanim` to false and [animstate](Animations/animstate.md) to `basestate`.

````cs
public void OverrideOver()
````

## Death and revival

These methods manages the concept of death and revival of an entity.

Starts the parameterless Death coroutine and store its result into `deathcoroutine`

````cs
public void StartDeath()
````

Starts Death with `activatekill` to true storing the result into `deathcoroutine`.

````cs
public IEnumerator Death()
````

Perform the death process of the entity setting `iskill` to `activatekill` at the end. More details at [Death](Notable%20methods/Death.md).

````cs
public IEnumerator Death(bool activatekill)
````

Revive an entity making it active again and undo its death.

````cs
public void Revive()
````

Specifically, this sets `iskill`, `dead` and `nocondition` to false as well as stop `deathcoroutine` if one was running. Then, LockRigid(false) is called with the `ccol` getting enabled with the center being set to `initialcenter` and its height/radius to the ones in `initialcolliderdata`. The world position of the entity is set to `startpos` + (0.0, 0.25, 0.0) with zero angles and `spin`. Finally, SetOverrides is called with all parameters to false which resets all overrides.

## Height adjustments

These method manages `height`, the visual offset of the entity's `sprite` from its `transform`'s origin.

Set `height` to `h`, `bobrange` to `range` and `bobspeed` to `spd` which will make be taken into effect on the next [UpdateHeight](Update%20process/UpdateHeight.md).

````cs
public void SetHeight(float h, float range, float spd)
````

Smoothly sets the `height` to `newhight` over the course of `frametime`.

````cs
public IEnumerator GradualHeight(float frametime)
````

````cs
public IEnumerator GradualHeight(float newheight, float frametime)
````

The default value of `newheight` is `initialheight`.

## Ice cube handling

These 2 methods setups the entity to be frozen in an ice cube and allows it to break it when it is no longer frozen. More details at [Freeze handling](Notable%20methods/Freeze%20handling.md)

````cs
public void Freeze()
````

````cs
public void BreakIce()
````

## Digging

Have the entity instantly dig underground without the first `digpart` by setting `digging` and `instdig` to true and `digtime` to 31.0.

````cs
public void InstantDig()
````

While the `spritetransform` scale's magnitude isn't 0.9 or higher, call ReturnFromAction and yield for a frame.

````cs
public IEnumerator StopDig()
````

## Follow

More details at [Follow](Notable%20methods/Follow.md)

````cs
private void Follow()
````

More details at [Follow](Notable%20methods/Follow.md#dofollow)

````cs
private void DoFollow()
````

Special logic to determine if the entity should follow `following` twice as close than usual.

````cs
private bool CloseMove()
````

Special movement logic for followers when the player is shielding.

````cs
private void ShieldMove(bool tempf)
````

## Sprite effects

These methods perform shaking and fading on the sprite.

For each `frametimer` frames, set the `spritetransform` local position to a random position between -`intensity` and `intensity` and restoring the position it had beforehand when `frametimer` frames elapsed.

````cs
public IEnumerator ShakeSprite(Vector3 intensity, float frametimer)
````

Calls the above overload where `frametimer` is passed through and the Vector3 `intensity` is a vector where each component is the float `intensity`

````cs
public IEnumerator ShakeSprite(float intensity, float frametimer)
````

Fade the `sprite` over `frametime` frames then yield a frame and if `destroy` is true, the entity is destroyed after this yield.

````cs
public IEnumerator FadeSprite(float frametime, bool destroy)
````

If `destroy` is true, this also calls LockRigid(true) and disables the `npcdata` if it was present after the fade is complete, but before the frame yield.

## Condition icons

These methods manages the conditions icons which are UI elements that reports details and presence of battle conditions.

Calls DestroyConditionIcons and then update `statusicons` and initialise `statusid` to 0 and the `statuscooldown` to 60.0 according to `data` positioned 0.5 units to the left of the `data.cursoroffset.x` if `right` is false or 0.5 units to the right if it is true.

````cs
public void UpdateConditionBubbles(bool right, MainManager.BattleData data)
````

This includes setting up the sprites of every conditions and medals icons during battle. The text inside these icons includes [SetText](../../SetText/SetText.md) calls. The method calls this one internally if `isplayer` with the player specific conditions and medals icons.

````cs
private Transform NewConditionIcon(Sprite csprite, Vector3 pos)
````

The icons are cycled by the help of [UpdateStatusIcons](Update%20process/UpdateStatusIcons.md).

If `statusicons` is present and not empty, every non null element is destroyed. `statusicons` is set to null once done.

````cs
public void DestroyConditionIcons()
````

This is called by MainManager.RemoveCondition. It determines the player or enemy asleep and numb status using MainManager.HasCondition. This method is very specific to RemoveCondition.

````cs
public void RefreshCondition()
````

## Late transform

Set the `latetrans` to null

````cs
public void StopLate()
````

## Cave of Trials

These methods manages materials and colors specific to the Cave of Trials when `cotunknown` is true

If `cotunknown` is true, set `spritebasecolor` to pure black fully opaque and if `extras` is present and non empty, `refreshedcotu` is set to true and all renderer and sprite render of every `extras` has they color set to pure black (half transparent for a renderer and fully opaque for sprite renderer).

````cs
public void RefreshCOT()
````

Set `spritebasecolor`, `sprite`'s color and the `sprite`'s material color to pure black fully opaque befoe calling RefreshCOT

````cs
public void ForceCOT()
````

## Dialogue bleep

If an `endata` for the [AnimID](../../Enums%20and%20IDs/AnimIDs.md) `animid` exists, set `dialoguebleepid` and `bleeppitch` from it unless `originalid` is -1 or below (None) in which case, they are set to 0 and 1.0 respectively.

````cs
public void SetDialogueBleep()
````

## Drop

More details at [Drop](Notable%20methods/Drop.md), this is a coroutine needed during battle to drop an airborne enemy.

````cs
public IEnumerator Drop()
````

## Scale change

Scales the entity to `target` smoothly over `frametime` frames. If `force` is false, `startscale` is what changes and if it's true, it's the `rotater`'s scale which makes it scale immediately without needing the handling of [UpdateFlip](Update%20process/UpdateFlip.md).

````cs
public IEnumerator ChangeScale(Vector3 target, float frametime, bool force)
````

## Trail

Reset the trail managed by [RefreshTrail](Update%20process/RefreshTrail.md). More information can be found there.

````cs
public void ResetTrail()
````

## Numb

A part of [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) that handles logic related to the numb animation. More details available there.

````cs
private void Numb()
````

## Return from action

This is a special procedure performed during StopDig and Follow after an action was performed to restore the entity behavior.

````cs
private void ReturnFromAction()
````

This first enables `shadow` if it wasn't. The, the `ccol` is enabled, the `rigid`'s isKinematic is set to false with gravity enabled, the `spritetransform`'s scale is set to a lerp between the current one and Vector3.one with a factor of 0.1 and set `spin` to zero if the switchcooldown expired. Finally, the fields `overrridejump`, `overrideanim` and `leiffly` are set to false.

## Check NEAR

If the `NEAR` [Modifiers](Modifiers.md) is present, this will destroy the entity if it is further away than 30.0 units from the player.

````cs
public void CheckNear()
````

## Miscellaneous

Returns true only if the entity has the `Follower`, `NPC`, `Enemy`, `PFollower` tag or `isplayer` or there is no `npcdata` or there is one being a PushRock or Item.

````cs
private bool CheckForCharacterEntity()
````

Sets the color of the Chompy's ribbon (stored in `extrasprites` 0) depending on [flag](../../Flags%20arrays/flags.md) 56 (the [Items](../../Enums%20and%20IDs/Items.md) id equipped to Chompy)

````cs
public void ChompyRibbon()
````

This is the special animation routine for Zasp. He will disappear with `appear` being false and appear if it is true

````cs
public IEnumerator ZaspWarp(bool appear)
````

Sets the world position of the entity to `pos`, yield for a frame and call the other overload passing `appear` storing the call into `specialanim` and then yield for a frame.

````cs
public IEnumerator ZaspWarp(bool appear, Vector3 pos)
````

This is the special animation for a [NPCControl](../NPCControl/NPCControl.md) of JumpSpring. The animation will mess with the `spritetransform` scale using the `squashammount` scale factor with the `squashspeed` multiplier over the course of `time` frames. If `gradual` is true, the `squashspeed` decreases over time.

````cs
public IEnumerator BounceAnim(float squashammount, float time, float squashspeed, bool gradual)
````

## Unused

These methods aren't called by anyone, but they still exists.

````cs
public void ActivateDefenseTap(float ammount)
````

````cs
private void LateGround()
````

````cs
public void SetLate(Transform obj, Vector3 pos)
````

````cs
public void FlipValue(float right)
````

````cs
public void ChangeAnimIfNotBattle(float id)
````

````cs
public IEnumerator LateVelocity(Vector3 ammount, float delay)
````

````cs
public IEnumerator LateVelocity(Vector3 ammount, float delay, float onlyifmagnitude, bool ignorey)
````

This one is called by the one above which makes it unused too

````cs
public IEnumerator LateVelocity(Vector3 ammount, float delay, float onlyifmagnitude, bool ignorey, int continuous)
````
