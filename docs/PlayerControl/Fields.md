# PlayerControl fields

## Public

|Name|Type|Description|
|---:|----|-----------|
|entity|EntityControl|The [EntityControl](../Entities/EntityControl/EntityControl.md) attached to the same GameObject as the PlayerControl, set by Start|
|beemerang|NPCControl||
|icecle|Transform||
|frozencube|Transform||
|npc|List<[NPCControl](../Entities/NPCControl/NPCControl.md)>|The list of map entities that the player is in interact range ordered by their distance to the player (ordering is updated by RefreshNPCs which is called by LateUpdate). The actual interact range check and appending of the map entities in the list is done by [NPCControl's LateUpdate](../Entities/NPCControl/LateUpdate.md)|
|lastpos|Vector3||
|switchcooldown|float||
|flycooldown|float|The amount of frames left during a flight before the action gets cancelled on its own. Reset to 240.0 on Start and once the action is cancelled and counted down during LateUpdate while `flying`|
|actionhold|float||
|flyheight|float||
|actioncooldown|float||
|iceclesize|float||
|pausecooldown|float||
|boulderbreak|float||
|movecd|float||
|interactcd|float||
|action|bool||
|digging|bool|Tells if the player is using `Beetle`'s hold action to dig underground|
|shield|bool|Tells if the player is being shielded by using `Moth`'s hold action which implicates that the `bubbleshield` should be visible as enforced by LateUpdate|
|lockkeys|bool||
|flying|bool|Tells if the player is flying using `Bee`'s hold action|
|startdig|bool||
|buttonhold|bool||
|canfly|bool||
|uproot|bool||
|setfolloweronground|bool||
|canpause=true|bool||
|candig|bool||
|submarine|bool||
|forceclosemove|bool||
|tattling|bool||
|ceiling|bool||
|dashing|bool||
|trueflip|bool||
|basespeed=5|int||
|jumproutine|Coroutine||
|standingon|Collider||
|lastdelta|Vector3||
|delta|Vector3||
|spd|Vector3||
|walkdelta|Vector3||
|lastaxis|Vector3||
|lastloadzone|Vector3||
|dashdelta|Vector3||
|conveyor|StaticModelAnim|???. This has `HideInInspector`, but since it's never assigned from prefab, it doesn't do anyting|

## private

|Name|Type|Description|
|---:|----|-----------|
|dashtarget|Vector3||
|tempcamoffset|Vector3?||
|digicon|SpriteRenderer[]|Should always contain 2 elements where both elements represents the UI icons to display when `digging` (first is the red arrow, second is `Beetle`'s icon). Both elements are initialised on Start|
|startheight|float?||
|tunnelpart|GameObject||
|diggingpart|GameObject||
|bubbleshield|GameObject|An instance of `Prefabs/Objects/BubbleShield` childed to the entity.`rotater` with a local position of (0.5, 1.25, 0.0) with an initial scale of Vector3.zero so it's not seen initially. All of this is initialised on Start and the visibility is done by adjusting its scale towards (3.0, 3.0, 2.0) or Vector3.zero on LateUpdate depending on the `shield` field|
|tbox|BoxCollider||
|ceildetect|GroundDetector||
|smoke|ParticleSystem||
|keepdig|float||
|idletime|float||
|footstep|float||
|respawncount|float||
|lastactionid|int||
|actionroutine|Coroutine||
