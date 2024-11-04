# PlayerControl fields

## Public

|Name|Type|Description|
|---:|----|-----------|
|entity|EntityControl|The [EntityControl](../Entities/EntityControl/EntityControl.md) attached to the same GameObject as the PlayerControl, set by Start|
|npc|List<[NPCControl](../Entities/NPCControl/NPCControl.md)>|The list of map entities that the player is in interact range ordered by their distance to the player (ordering is updated by RefreshNPCs which is called by LateUpdate). The actual interact range check and appending of the map entities in the list is done by [NPCControl's LateUpdate](../Entities/NPCControl/LateUpdate.md)|
|flycooldown|float|The amount of frames left during a flight before the action gets cancelled on its own. Reset to 240.0 on Start and once the action is cancelled and counted down during LateUpdate while `flying`|
|movecd|float|The amount of frames in a row where `delta` wasn't Vector3.zero updated in Update. The value won't go when it reaches 10.0 or above. It is only used as part of a failsafe in Hazard's HazardAction where if more than 3 `respawnretries` happened and the value is 10.0 or above (so it's maxed), all `playerdata` will have their position set to the `lastloadzone` + `MainCamera`'s forward vector * 0.1 * the `playerdata` index (this prevents z fighting)|
|digging|bool|Tells if the player is underground which means that entity.`sprite` local y position reached -1.5 or below, checked in FixedUpdate. TODO: toggled in DoActionTap if a submarine ???|
|shield|bool|Tells if the player is being shielded by using `Moth`'s hold action which implicates that the `bubbleshield` should be visible as enforced by LateUpdate|
|lockkeys|bool|If this is true, it prevents most form of inputs processing. Notably, GetInput, Movement and most of Update's regular movement inputs processing won't happen anymore except for DashBehavior and the assignement of `lastaxis` where these still can happen (`delta` stays to Vector3.zero)|
|flying|bool|Tells if the player is flying using `Bee`'s hold action|
|canpause|bool|Defaults to true. Being false prevents GetInput from processing any pause or HUD toggle inputs as well as prevent updates to `pausecooldown`, `idletime` and NPCControl.LateUpdate ??? TODO: recheck what this does|
|dashing|bool|Tells if the player is using `Beetle`'s tap action to dash|
|trueflip|bool|A value that is assigned during Update's regular movement inputs processing. It is assigned to false for holding input 2 (left) and to true for holding input 3 (right). The value is set to entity.`flip` when processing any tap action|
|basespeed|int|The default entity.`speed` when converted to float. This defaults to 5 and is never set again|
|lastdelta|Vector3|The last normalized, but unscaled direction the movement inputs leads to relative to the camera. This is computed on Update when regular movement input processing happens, but it is only assigned when `delta` didn't compute to a value of Vector3.zero so it's effectively the last direction the movement inputs where leading when they lead to somewhere. This is reset to be the camera's right or left vector normalized after processing a tap action|
|delta|Vector3|The last normalized and entity.`speed` scaled direction the movement inputs leads to relative to the camera. This is computed on every Update when regular movement input processing happens. It is effectively `lastdelta`, but constantly up to date and contains the actual magnitude as it is scaled by entity.`speed`|
|spd|Vector3|The last x/z velocity value to assign to the entity.`rigid` (the y component is ignored). This is both computed and used in Movement (called by LateUpdate)|
|walkdelta|Vector3|This is the x/z velocity values to assign to `spd` on the next Movement call (called by LateUpdate). It only applies when MainManager.`analog` isn't 0 TODO: ??? and is computed based on `delta` during Update|
|lastaxis|Vector3|The last x/z normalized direction that the analog horizontal + vertical inputs were observed to lead to (the y component is always assigned to 0.0). This is computed on Update during regular movement inputs processing or by DashBehavior when it is processing those inputs|
|switchcooldown|float|The amount of frames left before the current party switching concludes updated by LateUpdate. Set to 15.0 when SwitchOrder is called when a switch is initiated. When this reaches below 0.0, the switch is finalised and the value is set to -99999.0 to mark it as such. When this is above 0.0, it prevents some logic to happen such as some inputs processing in GetInput|
|actioncooldown|float|The amount of frames left before `tattiling` or ability inputs can be processed again. This is set when these inputs shouldn't be processed for a certain amount of time by GetInput. Updated by LateUpdate|
|action|bool|If this is true, it indicates that a DoActionTap is in progress when the player isn't a `submarine` and is only set to false when the action ends or when CancelAction is called. This field being true prevents the switch, jump/interact and abiliy inputs from being processed by GetInput|
|beemerang|NPCControl|The current [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) of the player if it exists which is only created during `Bee`'s DoActionTap|
|startdig|bool|This is set to true the moment a DoActionHold was detected to start the `digging` process, but it won't be completely registered until the entity.`sprite` local y position reaches -1.5 or below. When this is true, it prevents various logic to happen|
|submarine|bool|Tells if the player is currently in a submarine which affects many logic in the rest of the game including in PlayerControl such as entity rendering|
|pausecooldown|float|The amount of frames left before the pause inputs can be processed again, updated by LateUpdate when `canpause` is true. This can be set to prevent pausing for a certain amount of frames|
|tattling|bool|If true, it means that the player initiated the [SetText](../SetText/SetText.md) call to obtain help on `npc[0]` or the current map. It is set to false on LateUpdate when the [message](../SetText/Notable%20states.md#message) locks gets released|
|actionhold|float|The amount of frames since the ability input was detected as being held. This is used to detect which action to use between DoActionTap and DoActionHold. The latter applies if this field reaches 20.0 and the former applies if not, but still was held for at least a frame|
|uproot|bool|If true, it indicates that a [DigSpot](../Entities/NPCControl/ObjectTypes/DigSpot.md) will let the object hidden within to reveal itself on its OnTriggerStay with the player. It is set to true on GetInput when stopping to hold the ability input when `digging` was true (so undigging) and it is set to false in 0.1 seconds by CancelUproot which is invoked immediately when the field is set to true|
|canfly|bool|If true, it allows the `Bee`'s DoActionHold input to be processed. It is set to true on LateUpdate when the entity is `onground`|
|lastloadzone|Vector3||
|dashdelta|Vector3||
|icecle|Transform||
|frozencube|Transform||
|jumproutine|Coroutine||
|standingon|Collider||
|candig|bool||
|forceclosemove|bool||
|ceiling|bool||
|setfolloweronground|bool||
|interactcd|float||
|flyheight|float||
|iceclesize|float||
|boulderbreak|float||
|lastpos|Vector3||
|conveyor|StaticModelAnim|???. This has `HideInInspector`, but since it's never assigned from prefab, it doesn't do anyting|
|buttonhold|bool|TODO: ??? seemingly only set to true after being done with flying|

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
|keepdig|float|The amount of frames left to indicate PlayerControl that they should continue `digging` if one was initiated TODO: ??? detail stuff|
|idletime|float|The amount of frames in a row since `delta` had a value of Vector3.zero updated on Update. The value won't go higher when it reaches 1000.0 or above and it only has an effect once it gets above 250.0 where LateUpdate may reveal the main HUD and the money HUD|
|footstep|float||
|respawncount|float||
|lastactionid|int||
|actionroutine|Coroutine||
