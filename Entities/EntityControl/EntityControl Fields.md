# EntityControl Fields

This page aims to document the different fields available to EntityControl and their expected usage. Note: internal here means both EntityControl and NPCControl's concerns, not just EntityControl. A field being public doesn't necessarily mean it can safely be used externally. This document will clarify when a field can safely be set externally.

## Object hiearchy

These fields are intended to ease the navigation and accesses of the different key components and objects of the [Entity](../Entity.md). They can be as simple as avoiding a GetComponent call or just a way to verify the presence of an optional structure like a model. They are not intended to be set externally unless otherwise stated, but their properties can be set.

|Name|Type|Description|
|----|----|-----------|
|transform|Transform|The transform of the root object of the entity, assigned on [EntityControl Creation > CreateNewEntity](EntityControl%20Creation.md#createnewentity) and [Start](Start.md)|
|npcdata|NPCControl|The optional [NPCControl](../NPCControl/NPCControl.md) of the entity. This must be set externally or on [CreateEntities](CreateEntities.md)|
|ccol|CapsuleCollider|The capsule collider of the root object, assigned on [EntityControl Creation > CreateNewEntity](EntityControl%20Creation.md#createnewentity) and [Start](Start.md). The height and radius are loaded on [CreateEntities](CreateEntities.md)|
|rigid|RigidBody|The rigid body of the root object, assigned on [Start](Start.md)|
|rotater|Transform|The transform of the Rotater object which controls alignement, the first child of the root object, assigned on [Start](Start.md)|
|sprite|SpriteRenderer|The sprite renderer of the sprite object, assigned on [EntityControl Creation > CreateNewEntity](EntityControl%20Creation.md#createnewentity)|
|spritetransform|Transform|The transform of the Sprite object, a child of the Rotater object, assigned on [EntityControl Creation > CreateNewEntity](EntityControl%20Creation.md#createnewentity)|
|emoticon|Animator|The animator component of the Emoticon object which is an icon that appears at the top of the entity for interactions, a child of the Rotater object, assigned on [Start](Start.md)|
|emoticonsprite|SpriteRender|The sprite renderer component of the Emoticon object, a child of the Rotater object, assigned on [Start](Start.md)|
|moverotater|Transform|The transform of the MoveRotater object which is used for movement alignment, a child of the root object, assigned on [EntityControl Creation > CreateNewEntity](EntityControl%20Creation.md#createnewentity) and [Start](Start.md)|
|shadow|SpriteRender|The sprite renderer of the shadow object, a child of the root object. Assigned on [Start](Start.md) during CreateShadow|
|shadowtransform|Transform|The transform of the shadow object, a child of the root object. Assigned on [Start](Start.md) during CreateShadow|
|hpbar|Transform|The transform of the hp bar when applicable, a child of the root object. Assigned on CreateHPBar|
|hpbarfont|DynamicFont|A DynamicFont of the hpbar that shows the HP, Assigned on CreateHPBar|
|defstat|DynamicFont|A DynamicFont of the hpbar that shows the defense, Assigned on CreateHPBar|
|icecube|GameObject|The object of an ice cube when the entity is frozen, a child of the root object. Assigned on [Freeze handling > Freeze](Notable%20methods/Freeze%20handling.md#freeze)|
|movesmoke|Transform|The transform of an instance of `Prefabs/Particles/WalkDust` a child of the root object. Assigned when applicable on [LateStart](Notable%20methods/LateStart.md)|
|detect|BoxCollider|The BoxCollider of the Detector object with a RayDetector that is used to detect walls via hitwall, a child of the root object. Assigned by [EntityControl Methods > CreateDetector](EntityControl%20Methods.md#createdetector)|
|feet|GroundDetector|The GroundDetector of an instance of `Prefabs/GroundDetector` a child of the root object used to detect being on the ground via onground. Assigned on [EntityControl Methods > CreateFeet](EntityControl%20Methods.md#createfeet) with this entity as the parent|
|flowerbed|GameObject|An instance of `Prefabs/Particles/flowerbed`, a child of of the root object. Assigned during [OnTriggerStay](Update%20process/Unity%20events/OnTriggerStay.md) when the other collider has the `FlowerBed` tag|
|sound|AudioSource|The audio source of the root object, assigned on [Start](Start.md), but can be reassigned externally|

## Animations and [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md)

These fields manages the animations according to the animid.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|animid|int|Yes|The [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) of the entity, dictates the animation controller to use via the anim field. Can be set via [CreateEntities](CreateEntities.md)|
|originalid|int|No|The [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) at the start of [CheckSpecialID](Notable%20methods/CheckSpecialID.md) which may not be the same then the effective value from the animid field|
|animstate|int|Yes|The [animstate](Animations/animstate.md) of the entity, dictates the animation clip to play on the controller of the anim field, set via [CreateEntities](CreateEntities.md) if it is an item|
|basestate|int|Yes|The [animstate](Animations/animstate.md) to consider as the `Idle` one which overrides the default of 0, assigned during [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|walkstate|int|Yes|The [animstate](Animations/animstate.md) to consider as the `Walk` one which overrides the default of 1, assigned during [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|changedstate|bool|Yes|Determines if the [animstate](Animations/animstate.md) was changed externally and thus should bypass the landing on ground logic of [UpdateAirAnim](Update%20process/UpdateAirAnim.md) for all entities with an npcdata. This is only used in the [Anim](../../SetText/Commands/Individual%20commands/Anim.md) command.|
|animspeed|float|Yes|The time in second to transition between animations during [SetAnim](Animations/SetAnim.md)|
|anim|Animator|No|The animator the entity uses. This will use the controller of the animid field. This is first assigned on [Start](Start.md) or [AddModel](Notable%20methods/AddModel.md) , but it can be changed later via ForceAnimator (more details at [EntityControl Methods > Animator change](EntityControl%20Methods.md#animator-change))|
|hasiceanim|bool|No|Whether or not the entity has an ice animation clip present to play when the inice field is true. Assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|diganim|bool|No|Determines if this entity has animations when digging, assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|flyinganim|bool|Yes|Determines if the entity should be playing the flying version of the current [animstate](Animations/animstate.md)|
|forcefire|bool|No|Forces the fire variant of the entity's animid if it exist. This is only assigned to true during [Enemies data](../../TextAsset%20Data/Enemies%20data.md) loading for FireCape, FireWarden and FireKrawler. Enforced in [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|talking|bool|Yes|Determines if the entity is currently talking. This must be set externally to have an effect during [UpdateSprite](Update%20process/UpdateSprite.md)|
|notalk|bool|No|Determines if this entity should never have [SetAnim](Animations/SetAnim.md) called with a t (talking) argument assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when `endata`'s Object is true|
|bounceanim|Coroutine|Yes|Exposes a way to store the BounceAnim call to know if the entity is currently in a BounceAnim, set to null before a yield break or return|
|specialanim|Coroutine|Yes|Exposes a way to store a coroutine call to know if the entity is currently in a special animations requiring a coroutine, set to null before a yield break or return|

## Force move

These fields manages both the process of [EntityControl Methods > ForceMove](EntityControl%20Methods.md#forcemove) and [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards) which are considered force move as they are game controlled movement of the entity to a specific target.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|forcemoving|Coroutine|Yes|Exposes a way to store a [EntityControl Methods > ForceMove](EntityControl%20Methods.md#forcemove) call to know if the entity is currently in a ForceMove, set to null before a yield break or return|
|forcemove|bool|Yes|Determines if the entity is currently in a force move process via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards)|
|forcetarget|Vector3|Yes|The target position of a forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards)|
|forcemultiplier|float|No|The multiplier to the speed when moving during a forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards) if the caller specified one. This defaults to 1.0|
|forceanim|int|No|The [animstate](Animations/animstate.md) to use during the forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards) if the caller specified one. This defaults to walkstate|
|forcestop|int|No|The [animstate](Animations/animstate.md) to use when the forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards) is completed if the caller specified one. This defaults to basestate|
|forcetimer|float|No|The time remaining in frames before triggering the forcemove failsafe via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards), decreased on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) when applicable unless disabletimer is true|
|disabletimer|bool|Yes|Determines if the forcetimer should decrease on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) if applicable|
|looktowards|Vector3|No|The position the entity looks at during a forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards), null if not applicable|
|extratimer|bool|No|Determines if the forcetimer should be doubled when a forcemove is started which gives more time before triggering the failsafe. This can be assigned to true on Start with the `TIME` [Modifiers](Modifiers.md)|
|forcejump|bool|Yes|Determines if the entity should attempt to jump during a force move when hitwall and onground are true and jumpcooldown is expired. This must be set externally|
|ignorey|bool|Yes|Determines if the entity should ignore the y axis during the forcemove via [EntityControl Methods > MoveTowards](EntityControl%20Methods.md#movetowards) if the caller specified one.|

## Height management

These fields operate with the concept of height which is a visual vertical offset between the root `transform` and the `spritetransform`. This is accomplished by setting the local position of the latter upwards which won't affect physics, but affects visual rendering.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|height|float|Yes|The vertical offset of the spritetransform to the transform. Set to initialheight on [CreateEntities](CreateEntities.md), but overridden to minheight on [Start](Start.md) when applicable|
|minheight|float|Yes|The minimum height which is enforced on Start and in [CheckSpecialID](Notable%20methods/CheckSpecialID.md) where it is also assigned from `endata`|
|initialheight|float|Yes|The starting height, assigned to height on Start and on [CreateEntities](CreateEntities.md)|
|tempheightoverride|bool|No|This is only used during [Drop](Notable%20methods/Drop.md) when the entity is frozen in the air and therefore should drop to 0.0 instead of minheight|
|bobrange|float|Yes|The range of the oscillation done during [UpdateHeight](Update%20process/UpdateHeight.md) when the height is higher than 0.1, assigned on [CreateEntities](CreateEntities.md) or on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|bobspeed|float|Yes|The speed of the oscillation done during [UpdateHeight](Update%20process/UpdateHeight.md) when the height is higher than 0.1, assigned on [CreateEntities](CreateEntities.md) or on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|startbf|float|No|The initial bobrange, assigned to bobrange on [LateStart](Notable%20methods/LateStart.md) but also on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) and [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks) when applicable. The bobrange is restored to this on [Drop](Notable%20methods/Drop.md) and BreakIce, but it is possible to do so externally|
|startbs|float|No|The initial bobspeed, assigned to bobspeed on [LateStart](Notable%20methods/LateStart.md) but also on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) and [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks) when applicable. The bobspeed is restored to this on [Drop](Notable%20methods/Drop.md) and BreakIce, but it is possible to do so externally|

## Logic overrides

These fields allows to bypass common logic of the entity during its lifecycle. They act as toggles and most of them can be operated externally which allows to gain more control over the entity from outside EntityControl.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|overrideanim|bool|Yes|Determines if the entity should not have its [animstate](Animations/animstate.md) changed automatically which means only explicit writes to the animstate are allowed. This also prevents the sprite to be disabled and its sprite to be set to null when the animid becomes -1 (None). Assigned to true on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) if it's a `KeyL`, `KeyR` or `Tablet`|
|overridemovesmoke|bool|Yes|Determines if the movesmoke should never be positioned at the root object during [UpdateMoveSmoke](Update%20process/UpdateMoveSmoke.md). Set to true on Start and [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|overrideshadow|bool|Yes|Forces the creation and updates of shadow as long as iskill is false, assigned in [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata` or to true when applicable. This needs to be set as part of the startup process or shadow will never be created|
|overrideanimfunc|bool|Yes|Determines if the entity should not be affected by a call to AnimationFunctions.ExtraAnims, assigned to true on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) if it's a `KeyL`, `KeyR` or `Tablet`|
|overrridejump|bool|Yes|Determines if the entity should not have its [animstate](Animations/animstate.md) changed to `Jump` or `Fall` automatically during [UpdateAirAnim](Update%20process/UpdateAirAnim.md), assigned at several places internally notably on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|overrideflip|bool|Yes|Determines if the entity should skip any [UpdateFlip](Update%20process/UpdateFlip.md) cycle|
|overrideonlyflip|bool|Yes|Determines if the entity should skip regular flip handling in the [UpdateFlip](Update%20process/UpdateFlip.md) cycle (this means any updates other than digging when spin is zero)|
|overrideheight|bool|Yes|Determines if the entity should skip any [UpdateHeight](Update%20process/UpdateHeight.md) cycle even if it isn't the player and not an npcdata of type Object|
|overrideanimspeed|bool|Yes|Determines if the standard animation speed logic should be bypassed on Update, must be set externally or by [EntityControl Methods > Overrides](EntityControl%20Methods.md#overrides)|
|overridefollow|bool|Yes|Determines if the entity should not be controlled by the \[\[Entities/EntityControl/Follow|
|overrideshieldpos|Vector3?|Yes|Overrides the default bubbleshield position if assigned. This must be set externally|
|overridefly|bool|No|Determines if the entity will never attempt to play a fly animation or receive [UpdateAirAnim](Update%20process/UpdateAirAnim.md) cycles, assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|

## Extras and optional objects

These fields manages optional objects that the entity maintains. They are usually created and used on a per [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) needs.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|extras|GameObject|No|An array of extra objects to keep track of. Assigned with a variable length on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) and during the [AnimSpecific](Animations/AnimSpecific.md) updates|
|spinextra|Vector3\[\]|No|An array of extra angles to add to the extras and extralines assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) and [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks) when applicable|
|extraanims|Animator\[\]|No|An array of extra animator to use which is required for certain [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md). Assigned if applicable on [AnimSpecific > UpdateAnimSpecific](Animations/AnimSpecific.md#updateanimspecific)|
|extrasprites|SprtiteRender\[\]|No|An array of extra sprite render to use which is required for certain [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md). Assigned if applicable on [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks)|
|extralines|LineRenderer\[\]|No|A set of extra lines the entity can use, only used for DeadLanderB during [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks)|
|line|LineRenderer|No|The line renderer of the Line object, childed to the parent sent to AddLine or to the spritetransform if null. AddLine must be called to have this assigned which it is on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) for RizGrandpa|
|subentity|EntityControl\[\]|No|An array of sub entities that this entity has access to. This is specifically used for SeedlingKing|
|firepart|Transform|Yes|This is not used internally and is left to null. This must be set and used externally|

## Ice

These fields manages any ice related event, particularly during [Freeze handling](Notable%20methods/Freeze%20handling.md).

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|inice|bool|Yes|Determines if the entity is affected by an ice related event, set to true on [Start](Start.md) if the current map's `icemap` is true|
|icecubeprefab|GameObject|No|An instance of `Prefabs/Objects/icecube`, assigned on [Start](Start.md)|
|nofallfrozen|bool|No|Determines if the entity should stay in place when [Drop](Notable%20methods/Drop.md) occurs while the entity is frozen|
|freezesize|Vector3|No|The size of the ice cube when the entity is frozen, assigned on [CreateEntities](CreateEntities.md), [Enemies data](../../TextAsset%20Data/Enemies%20data.md) loading or [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|freezeoffset|Vector3|No|The offset of the ice cube when the entity is frozen, assigned on [CreateEntities](CreateEntities.md), [Enemies data](../../TextAsset%20Data/Enemies%20data.md) loading or [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|initialfrezeoffset|Vector3|No|The initial offset of the ice cube when the entity is frozen, assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) to freezeoffset|
|shakeice|bool|Yes|Determines if the icecube should shake when being rendered if applicable|

## Rendering and alignment

These fields changes the way the entity is rendered or aligned horizontally.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|speed|float|Yes|The movement speed of the entity, assigned on [CreateEntities](CreateEntities.md) or during [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable. This defaults to 5.0|
|flip|bool|Yes|Whether the sprite should be rendered flipped by 180 degrees on the y axis, must be set externally, by [CreateEntities](CreateEntities.md) or if needed internally|
|alwaysflip|bool|No|Determines if [UpdateFlip](Update%20process/UpdateFlip.md) should be called each [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) even without an incamera update, assigned to true on Start with the `ALF` [Modifiers](Modifiers.md)|
|spin|Vector3|Yes|The rotation in angles to apply on the next [UpdateFlip](Update%20process/UpdateFlip.md) when not digging and overrideflip is false, assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) for a CrystalBerry|
|lockrotater|bool|No|Determines whether to lock the y angle of the rotater object instead of letting it rotate to the main camera y angle, assigned to true on Start with the `ROT` [Modifiers](Modifiers.md)|
|spritebasecolor|Color|Yes|The suggested color to render the sprite, assigned to fully opaque white on [EntityControl Creation](EntityControl%20Creation.md)|
|hologram|bool|Yes|Determines if the entity should be rendered as a hologram. This can be assigned to true on Start with the `Holo` or `COT` [Modifiers](Modifiers.md)|
|cotunknown|bool|Yes|Determines if the entity should be rendered as a silhouette during the Cave of Trials. This can be assigned to true on Start with the `COT` [Modifiers](Modifiers.md)|
|refreshedcotu|bool|No|Indicates that the entity processed a RefreshCOT call|
|stopspinonground|bool|Yes|A fire and forget flag to stop the spin on the next [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|
|startscale|Vector3|Yes|The base scale of the entity. truescale will be set to this during [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) which will cause the spritetransform to be scaled by it during [UpdateFlip](Update%20process/UpdateFlip.md). The default is Vector3.one, writing to it will change the scale on the next [UpdateFlip](Update%20process/UpdateFlip.md)|
|truescale|Vector3|No|The Vector3 that will be used to scale spritetransform on the next [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) using the startscale value. Initially set on [LateStart](Notable%20methods/LateStart.md) to startscale|
|modelscale|Vector3|No|The scale of the model if applicable, assigned on [AddModel](Notable%20methods/AddModel.md) to the model's actual scale. This is never set once startup finishes.|
|shrink|bool|Yes|Determines if the entity should be progressively shrunk on [UpdateVelocity](Update%20process/UpdateVelocity.md) by setting startscale towards zero|
|nomodel|bool|No|Determines if this entity should be considred to not have a model and to not attempt to load any, only set to true on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) if it's a `KeyL`, `KeyR` or `Tablet`|
|trail|bool|Yes|Determines if the entity has a trail which are rendered copies of the entity with half transparency as they move|
|traildata|TrailData|No|The data of the trails if applicable, managed by [RefreshTrail](Update%20process/RefreshTrail.md)|

## Physics and movement handling

These fields manages any physics related tasks, particularly movement, but also collisions.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|startvelocity|Vector3|No|The starting velocity of the rigid, assigned on [Start](Start.md)|
|deltavelocity|Vector3|No|The last velocity observed since the last \[\[Entities/EntityControl/Move|
|startpos|Vector3|Yes|The last starting position of the entity. Initially set on [CreateEntities](CreateEntities.md) or to the transform's position on Start or to the point a raycast will hit from the transform heading down with the `COG` [Modifiers](Modifiers.md) or on [LateStart](Notable%20methods/LateStart.md) to the transform position if it still isn't set. This is also set on SetPosition and it is used to restore the transform position if needed, but it can be done externally|
|onground|bool|Yes|The result of the feet's GroundDetector which determines if the entity is on the ground. It it can be forced externally to force the result. The default value is true, but it is overridden to false on Start with the `NGS` [Modifiers](Modifiers.md)|
|hitwall|bool|Yes|The result of the detect's RayDetector which determines if the entity has hit a wall, but it can be set externally to force the result.|
|walktype|WalkType|No|The type of walking of this entity during \[\[Entities/EntityControl/Move|
|jumpheight|float|Yes|The height of a [EntityControl Methods > Jump](EntityControl%20Methods.md#jump) for this entity. Also used to clamp the y velocity in [UpdateVelocity](Update%20process/UpdateVelocity.md), set to 10.0 by default|
|jumpcooldown|float|No|The time in frames to wait before the entity is allowed to jump again since the last [EntityControl Methods > Jump](EntityControl%20Methods.md#jump), decreased towards 0 by [UpdateGround](Update%20process/UpdateGround.md)|
|springcooldown|bool|No|Determines if the entity is currently in ascension after touching a JumpSpring|
|offgroundframes|float|Yes|The amount of time in frames this entity observed onground being false, updated on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md), but reset to 0.0 if it reaches 1000.0|
|initialcenter|Vector3|No|The center of the ccol initially when [Start](Start.md) occurs|
|initialcolliderdata|Vector2|No|The height and radius (in x and y respectively) of the ccol when [Start](Start.md) occurs|
|fixedentity|bool|Yes|Determines if this entity should be fixed in position, assigned to true on [Start](Start.md) if the entity is a TestSign or with the `Fixed` or `FxdCol` [Modifiers](Modifiers.md)|
|ignorewater|bool|No|Determines whether this entity is not affected by water Hazards, assigned when applicable on [CheckSpecialID](Notable%20methods/CheckSpecialID.md)|
|leiffly|bool|No|Determines if the entity is Moth and he is currently in his flying animation. Used during [FixedUpdate](Update%20process/Unity%20events/FixedUpdate.md) to set his position as he flies|

## Previous values

These fields stores the previous values of other fields observed at particular points in the entity lifecycle. Most of them shouldn't be set externally, but some can be set to force some kind of update.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|oldid|int|No|The last animid observed since the last animation updates of [UpdateSprite](Update%20process/UpdateSprite.md)|
|laststate|string|No|The last effective [animstate](Animations/animstate.md) played including the arguments of [SetAnim](Animations/SetAnim.md). Set only when playing an animation from [SetAnim](Animations/SetAnim.md)|
|lastvelocity|Vector3|No|The velocity last observed when pausing happened on [Update](Update%20process/Unity%20events/Update.md) and then used as the rigid's velocity on [Update](Update%20process/Unity%20events/Update.md) after unpause before going back to null|
|lastgravity|bool|No|The gravity's enabling that was last observed when pausing happened on [Update](Update%20process/Unity%20events/Update.md) and then used as the rigid's useGravity on [Update](Update%20process/Unity%20events/Update.md) after unpause|
|lastvolume|float|No|The sound's volume that was last observed when pausing happened on [Update](Update%20process/Unity%20events/Update.md) and then used as the sound's volume on [Update](Update%20process/Unity%20events/Update.md) after unpause|
|pausepos|Vector3|No|The position last observed when pausing happened on [Update](Update%20process/Unity%20events/Update.md) and set to null when unpausing happened on [Update](Update%20process/Unity%20events/Update.md). Used to lock the position when paused unless activeonpause is true, we are in an event, iskill is true or the entity is dead|
|lastshadow|Vector3|No|The last position of the transform when [RefreshShadow](Update%20process/RefreshShadow.md) updated the shadow|
|oldback|bool|No|The last backsprite observed since the last animation updates of [UpdateSprite](Update%20process/UpdateSprite.md)|
|oldttalk|bool|No|The last talking observed since the last animation updates of [UpdateSprite](Update%20process/UpdateSprite.md)|
|lastice|bool|No|The last inice observed since the last animation updates of [UpdateSprite](Update%20process/UpdateSprite.md)|
|oldstate|int|Yes|The last [animstate](Animations/animstate.md) observed by the last animation update of [UpdateSprite](Update%20process/UpdateSprite.md)|
|lastpos|Vector3|Yes|The last transform's position observed since the last [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) assigned to startpos on [CreateEntities](CreateEntities.md) or on Start|
|oldfly|bool|Yes|The last flyinganim observed since the last animation updates of [UpdateSprite](Update%20process/UpdateSprite.md)|
|oldtalk|bool|Yes|The last value of talking observed during the last [UpdateSprite](Update%20process/UpdateSprite.md)|

## Digging

These fields manages the process of digging underground.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|digging|bool|Yes|Determines if the entity is digging underground which is used on several update methods.|
|digtime|float|Yes|The time in frames since the entity started digging underground|
|digscale|Vector3|Yes|The scale to render digpart\[1\] which is the digging object when fully underground. The default is Vector3.one|
|digpart|GameObject\[\]|No|An array of 2 objects which are both children of the root object. The first is an instance of `Prefabs/Particles/DirtFlying` when starting a dig and the second is an instance of `Prefabs/Particles/Digging` when fully digging. Assigned to a length of 2 on [Start](Start.md)|
|nodigpart|bool|No|Determines if digpart should be created and handled while digging during [UpdateFlip](Update%20process/UpdateFlip.md). Set to true during \[\[Entities/EntityControl/Follow|
|instdig|bool|No|Determines if the entity should immediately start digging??? TODO: this might not be practically used because the only possible usage implies a throw during UpdateFlip|

## Identification

These fields provides more information about who this entity is.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|isplayer|bool|No|Determine if this is the player entity with the PlayerControl, assigned to true on [LateStart](Notable%20methods/LateStart.md) if the entity has the `Player` tag|
|mainparty|bool|Yes|Determines if the entity is part of the main party when created which means the party members that follow the player excluding the player itself. Must be set externally and is only used in CloseMove during a \[\[Entities/EntityControl/Follow|
|playerentity|bool|No|Determines if this entity is related or is the player, assigned to true on [Start](Start.md) if the entity has the `Player` or `PFollower` tag|
|isfollower|bool|No|Determines if the entity is a follower, assigned on [LateStart](Notable%20methods/LateStart.md) when the entity has the `PFollower` tag|
|tempfollower|bool|No|Determines if the entity is a temporary follower, set to true when the entity was created using [AddFollower](../../SetText/Commands/Individual%20commands/Addfollower.md)|
|battle|bool|Yes|Determines if this is a battle entity, must be set externally or during [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|battleid|int|Yes|The index of the entity in the current battle's enemydata, -1 when not applicable|

## Emoticon and status icons

These fields manages both the emoticon to show in `emoticon` or the status condition / medal icon to show during battle.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|emoticonid|int|Yes|The id of the emoticon to play on the next [UpdateEmoticon](Update%20process/UpdateEmoticon.md) when emoticoncooldown hasn't expired yet|
|emoticonoffset|Vector3|Yes|The offset to render the emoticon from the height during [UpdateEmoticon](Update%20process/UpdateEmoticon.md), assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`'s freezeflipoffset when applicable and on [CreateEntities](CreateEntities.md)|
|emoticoncooldown|float|Yes|The period in frames to listen for emoticon changes and play the one in emoticonid during [UpdateEmoticon](Update%20process/UpdateEmoticon.md)|
|statusid|int|No|The id of the current status icon to show from statusicons until statuscooldown expires|
|statusicons|Transform\[\]|Yes|The status transforms of the status icons attached to the entity|
|statuscooldown|float|No|The time in frames left to display the statusid current status icon from statusicons|
|nocondition|bool|Yes|Determines if the entity shouldn't render any condition icons during [UpdateStatusIcons](Update%20process/UpdateStatusIcons.md). Set to true at the start of [Death](Notable%20methods/Death.md) and to false in Revive|

## Follow

These fields configures the [Follow](Notable%20methods/Follow.md) system and allow access to the follow chain.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|following|EntityControl|Yes|The [Entity](../Entity.md) to follow which involves physically moving it such that it follows it during \[\[Entities/EntityControl/Follow|
|followedby|Transform|Yes|The entity that follows this one, null if not followed by anyone. This must be set externally which is typically by MainManager|
|followoffset|float|Yes|The distance to be away from the following entity|
|tempfollowerid|int|Yes|This is specific to Chompy to determine who she follows. This must be set externally|

## Camera active range info

These fields manages the concept of an entity being "active" which means they receive more attention during the update cycle. This is typically due to being in the range of the camera, but it can be forced on.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|incamera|bool|No|Determines if the entity is in the main camera range, updated every 3 frames on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md), stays true when alwaysactive is true|
|camdistance|float|No|The distance from the entity to the main camera, updated every 3 frames on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|
|campos|Vector3|No|The viewport position of the entity from the main camera, updated every 3 frames on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|
|alwaysactive|bool|Yes|Determines if this entity should always be considered active by overriding incamera to true during UpdateCamPos (called by [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)). This also forces the forcetimer to decrease and [UpdateHeight](Update%20process/UpdateHeight.md) to be called from [LateUpdate](Update%20process/Unity%20events/LateUpdate.md) when they apply. Assigned to true on [Start](Start.md) with the `ALW` [Modifiers](Modifiers.md) or during [CheckSpecialID](Notable%20methods/CheckSpecialID.md) when applicable|
|activeonpause|bool|Yes|Determines if this entity should receive updates and late updates when paused, minipaused, [message](../../SetText/Global%20vars%20used/message.md) lock being active or dead, assigned to true on [Start](Start.md) with the `PAU` [Modifiers](Modifiers.md)|

## Shadow

These fields handles the `shadow` object.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|hasshadow|bool|Yes|Determines if shadows should be rendered for this entity, assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) to true or overridden from `endata` / false if applicable. If this was false by the end of [CheckSpecialID](Notable%20methods/CheckSpecialID.md), the shadow will never be created|
|shadowsize|float|Yes|The maximum base scale of the shadow object assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|

## Sound

These fields configure the sounds to play using `sound`.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|soundonpause|bool|No|Determines if the entity should be muted when it goes out of range or the game is paused, assigned by [NPCControl](../NPCControl/NPCControl.md)'s `SetUp` when it is a Geizer|
|soundvolume|float|No|The volume to play the next sounds clips from the sound AudioSource. PlaySound must be called to set this|

## Death

These fields manages the concept of [Death](Notable%20methods/Death.md) and also configures it when it occur.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|dead|bool|Yes|Determines if the entity is currently dead which is assigned to true at the start of [Death](Notable%20methods/Death.md).|
|iskill|bool|Yes|Determines if the entity is killed, similar to death, but is only optionally set during [Death](Notable%20methods/Death.md) when requested.|
|spitexp|int|Yes|The amount of EXP orb to drop when [Death](Notable%20methods/Death.md) occur. This must be set externally|
|spitmoney|int|Yes|The amount of berries to drop when [Death](Notable%20methods/Death.md) occur. This must be set externally|

## Item

These fields are specific to item entities.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|item|bool|No|Determines if this entity is an item. This must be set internally when applicable or on [CreateEntities](CreateEntities.md) if it is an item|
|showitem|bool|No|Indicates that this entity must render an item sprite. This can be set to true on Start using the `shwKEY` [Modifiers](Modifiers.md)|
|itemstate|int|Yes|The [Items](../../Enums%20and%20IDs/Items.md) or [Medal](../../Enums%20and%20IDs/Medal.md) id, only applicable when item is true|
|destroytype|DeathType|Yes|The type of death process used for [Death](Notable%20methods/Death.md). This is normally loaded from \[\[TextAsset Data/Entity data#`entitydata` directory|

## Drop

These fields are related to the [Drop](Notable%20methods/Drop.md) procedure.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|shakeondrop|bool|No|Determines if the entity should cause a screen shake, a sound and smoke effects to happen when [Drop](Notable%20methods/Drop.md) has dropped the entity to the ground, Assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`|
|droproutine|Coroutine|Yes|Exposes a way to store the [Drop](Notable%20methods/Drop.md) call to know if the entity is dropping, set to null before a yield break or return|

## Bubble shield

These fields are related to the use of the bubble shield.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|shieldenabled|bool|Yes|Determines if the entity has an active bubble shield which will grow bubbleshield to its scale. This must be set externally|
|bubbleshield|DialogueAnim|No|The DialogueAnim of an instance of `Prefabs/Objects/BubbleShield` from the root of the asset tree childed to the Rotater object when battle and isplayer are both true on Start|

## Dialogue bleep

These fields dictate the default sound to use during [SetText](../../SetText/SetText.md)'s bleep and its [Bleep](../../SetText/Commands/Individual%20commands/Bleep.md) command.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|dialoguebleepid|int|Yes|The bleep sound id to use in the [Bleep](../../SetText/Commands/Individual%20commands/Bleep.md) [SetText](../../SetText/SetText.md) command, assigned from `endata` on [Start](Start.md) during SetDialogueBleep|
|bleeppitch|int|Yes|The bleep pitch to use in the [Bleep](../../SetText/Commands/Individual%20commands/Bleep.md) [SetText](../../SetText/SetText.md) command, assigned from `endata` on [Start](Start.md) during SetDialogueBleep|

## Late transform

The late transform is a special transform the entity has available that lives completely separately to the regular update cycle. These fields manages that concept.

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|latetrans|Transform|Yes|A transform that gets positioned by latepos every [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|
|latepos|Vector3|Yes|The position to set latetrans on [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|

## Miscellaneous

|Name|Type|External set?|Description|
|----|----|-------------|-----------|
|originalmap|Transform|No|The transform of the current map observed on [LateStart](Notable%20methods/LateStart.md)|
|setup|bool|No|Determines if [LateStart](Notable%20methods/LateStart.md) has executed. Assigned to true at the end of it.|
|icooldown|float|Yes|The period in frames left to the invulnerability|
|noclock|bool|Yes|Related to PlatformNoClock ??? This must be set externally|

## Used only by [NPCControl](../NPCControl/NPCControl.md)

These fields are declared in [EntityControl Creation](EntityControl%20Creation.md), but aren't actually used there outside of declaration with optional initialization.They are only useful in [NPCControl](../NPCControl/NPCControl.md), only what happens on [EntityControl Creation](EntityControl%20Creation.md) is documented here.

|Name|Type|Description|
|----|----|-----------|
|hideinside|bool|Assigned to true on Start with the `HIDE` [Modifiers](Modifiers.md)|
|alwaysemoticon|bool|Assigned to true on Start with the `ShwEm` [Modifiers](Modifiers.md)|
|spawnpoint|Vector3|Set to the transform's position on Start|
|overrideminheight|bool|Determines if the entity should not have minheight affect its height. This is only used by [NPCControl](../NPCControl/NPCControl.md)|
|activeinevents|bool|Left to default (false)|
|killonfall|bool|Left to default (false)|

## Practically unused fields

These fields are present, but they do not have any practical uses in the game. They can still impact logic if they are changed externally or they are used as constant value meaning they should never be changed.

|Name|Type|Description|
|----|----|-----------|
|noemoticon|bool|This can allow to opt out of having an emoticon object if set to true on Start. The field is functional, but never set|
|preloadedobjects|GameObject\[\]|Contains the array of preloaded objects assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`. This field's sole purpose is to cache ressources loading for later|
|preloadedsprites|Sprite\[\]|Contains the array of preloaded sprites assigned on [CheckSpecialID](Notable%20methods/CheckSpecialID.md) from `endata`. This field's sole purpose is to cache ressources loading for later|
|oldground|bool|The last onground value observed since the last [LateUpdate](Update%20process/Unity%20events/LateUpdate.md)|
|mapentity|bool|Presumably, this would indicate that the entity is from a map, but this is covered by npcdata not being null|
|flipspeed|float|The speed to flip the angles of the sprite during a regular flip handling of [UpdateFlip](Update%20process/UpdateFlip.md). This is left to its default value which is 0.2|
|extraoffset|Vector3|An additional offset to apply on [UpdateHeight](Update%20process/UpdateHeight.md), [UpdateEmoticon](Update%20process/UpdateEmoticon.md) and the ShakeSprite Coroutine. This is left to its default value of Vector3.zero|
|speedbuffer|float\[\]|This is only used for DeadLanderB during [AnimSpecific > AnimSpecificQuirks](Animations/AnimSpecific.md#animspecificquirks) with a value always being `{ 0.5, 0.34, 0.15, 0.3, 0.6 }`|
|followlimit|float|The maximum square distance in units of a \[\[Entities/EntityControl/Follow|
|followdistance|float|The minimum square distance in units of a \[\[Entities/EntityControl/Follow|
|followjump|float|The minimum square distance in units of a \[\[Entities/EntityControl/Follow|
|soundidstance|float|The maximum distance in units where the new volume of sound during [UpdateSound](Update%20process/UpdateSound.md) will be 0 when reached or exceeded from the camdistance. This is left to the default value of 25.0|
|lockback|bool|Determines if the entity shouldn't change its backsprite on FaceTowards unless it is forced. This is functional, but never set to true|
|usebuffer|bool|This is never set to true, but it has an unknown usage in DoFollow|
