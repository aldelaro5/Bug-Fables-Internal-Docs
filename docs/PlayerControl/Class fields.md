# PlayerControl fields
This page documents every class fields of PlayerControl.

It features a mix of public and private fields.

## Public
Since there are many public fields, they will be split into categories.

## Essentials

|Name|Type|Description|
|---:|----|-----------|
|entity|EntityControl|The [EntityControl](../Entities/EntityControl/EntityControl.md) attached to the same GameObject as the PlayerControl, set by Start|

## Field abilities specifics

|Name|Type|Description|
|---:|----|-----------|
|beemerang|NPCControl|The current [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) of the player if it exists which is only created during `Bee`'s DoActionTap|
|flycooldown|float|The amount of frames left during a flight before the action gets cancelled on its own. Reset to 240.0 on Start and once the action is cancelled and counted down during LateUpdate while `flying`|
|digging|bool|Tells if the player is underground which means that entity.`sprite` local y position reached -1.5 or below, checked in FixedUpdate. If the player is in a `submarine`, the semantic changes: this field now tells if the player is underwater or not, but all of the effects of `digging` applies|
|shield|bool|Tells if the player is being shielded by using `Moth`'s hold action which implicates that the `bubbleshield` should be visible as enforced by LateUpdate|
|dashing|bool|Tells if the player is using `Beetle`'s tap action to dash|
|flying|bool|Tells if the player is flying using `Bee`'s hold action|
|startdig|bool|This is set to true the moment a DoActionHold was detected to start the `digging` process, but it won't be completely registered until the entity.`sprite` local y position reaches -1.5 or below. When this is true, it prevents various logic to happen|
|icecle|Transform|An instance of `Prefabs/Objects/icecle` when the double tap action of `Moth` is used as it will be dropped, then destroyed|
|iceclesize|float|The scale of the `icicle`. It is only used and updated during `Moth` double tap action when it progressively goes from 0.0 to 1.0 in the course of ~20.0 frames to reveal the `icicle`|
|uproot|bool|If true, it indicates that a [DigSpot](../Entities/NPCControl/ObjectTypes/DigSpot.md) will let the object hidden within to reveal itself on its OnTriggerStay with the player. It is set to true on GetInput when stopping to hold the ability input when `digging` was true (so undigging) and it is set to false in 0.1 seconds by CancelUproot which is invoked immediately when the field is set to true|

## Movement

|Name|Type|Description|
|---:|----|-----------|
|movecd|float|The amount of frames in a row where `delta` wasn't Vector3.zero updated in Update. The value won't go when it reaches 10.0 or above. It is only used as part of a failsafe in Hazard's HazardAction where if more than 3 `respawnretries` happened and the value is 10.0 or above (so it's maxed), all `playerdata` will have their position set to the `lastloadzone` + `MainCamera`'s forward vector * 0.1 * the `playerdata` index (this prevents z fighting)|
|basespeed|int|The default entity.`speed` when converted to float. This defaults to 5 and is never set again|
|lastdelta|Vector3|The last normalized, but unscaled direction the movement inputs leads to relative to the camera. This is computed on Update when regular movement input processing happens, but it is only assigned when `delta` didn't compute to a value of Vector3.zero so it's effectively the last direction the movement inputs where leading when they lead to somewhere. This is reset to be the camera's right or left vector normalized after processing a tap action|
|delta|Vector3|The last normalized and entity.`speed` scaled direction the movement inputs leads to relative to the camera. This is computed on every Update when regular movement input processing happens. It is effectively `lastdelta`, but constantly up to date and contains the actual magnitude as it is scaled by entity.`speed`|
|spd|Vector3|The last x/z velocity value to assign to the entity.`rigid` (the y component is ignored). This is both computed and used in Movement (called by LateUpdate)|
|walkdelta|Vector3|This is the x/z velocity values to assign to `spd` on the next Movement call (called by LateUpdate). It only applies when MainManager.`analog` isn't 0 (analog sensitivity isn't OFF) and is computed based on `delta` during Update|
|lastaxis|Vector3|The last x/z normalized direction that the analog horizontal + vertical inputs were observed to lead to (the y component is always assigned to 0.0). This is computed on Update during regular movement inputs processing or by DashBehavior when it is processing those inputs|
|dashdelta|Vector3|The last movement vector used to move `Beetle` during a DashBehavior. This is computed by a lerp from the existing value to `dashtarget`.normalized * 4.0 which is determined by reading movement inputs (the lerp factor is `framestep` * 0.025)|
|forceclosemove|bool|If this is true, it forces the [CloseMove](../MapControl/Follower%20system.md#closemove) logic to apply to every follower when the player isn't `dashing`. This is only set to true when colliding with a [TempPlatform](../Entities/NPCControl/ObjectTypes/TempPlatform.md)|

## Inputs and state processing

|Name|Type|Description|
|---:|----|-----------|
|npc|List<[NPCControl](../Entities/NPCControl/NPCControl.md)>|The list of map entities that the player is in interact range ordered by their distance to the player (ordering is updated by RefreshNPCs which is called by LateUpdate). The actual interact range check and appending of the map entities in the list is done by [NPCControl's LateUpdate](../Entities/NPCControl/LateUpdate.md)|
|lockkeys|bool|If this is true, it prevents most form of inputs processing. Notably, GetInput, Movement and most of Update's regular movement inputs processing won't happen anymore except for DashBehavior and the assignement of `lastaxis` where these still can happen (`delta` stays to Vector3.zero)|
|canpause|bool|Defaults to true. Being false prevents GetInput from processing any pause or HUD toggle inputs as well as prevent updates to `pausecooldown`, `idletime` and prevents any [NPC](../Entities/NPCControl/NPC.md) to have [CheckEmoteFlag](../Entities/NPCControl/Notable%20methods/CheckEmoteFlag.md) called on them to update their emote|
|switchcooldown|float|The amount of frames left before the current party switching concludes updated by LateUpdate. Set to 15.0 when SwitchOrder is called when a switch is initiated. When this reaches below 0.0, the switch is finalised and the value is set to -99999.0 to mark it as such. When this is above 0.0, it prevents some logic to happen such as some inputs processing in GetInput|
|actioncooldown|float|The amount of frames left before `tattiling` or ability inputs can be processed again. This is set when these inputs shouldn't be processed for a certain amount of time by GetInput. Updated by LateUpdate|
|action|bool|If this is true, it indicates that a DoActionTap is in progress when the player isn't a `submarine` and is only set to false when the action ends or when CancelAction is called. This field being true prevents the switch, jump/interact and abiliy inputs from being processed by GetInput|
|pausecooldown|float|The amount of frames left before the pause inputs can be processed again, updated by LateUpdate when `canpause` is true. This can be set to prevent pausing for a certain amount of frames|
|actionhold|float|The amount of frames since the ability input was detected as being held. This is used to detect which action to use between DoActionTap and DoActionHold. The latter applies if this field reaches 20.0 and the former applies if not, but still was held for at least a frame|
|canfly|bool|If true, it allows the `Bee`'s DoActionHold input to be processed. It is set to true on LateUpdate when the entity is `onground`|
|interactcd|float|The amount of frames left before [Interact](../Entities/NPCControl/Notable%20methods/Interact.md) can be called again as a result of [GetInput](GetInput.md)'s input processing. This is only set to 15.0 on EndEvent when `lastevent` wasn't 3 (the cooking event). The cooldown is updated on every LateUpdate cycle that's not in a `pause` or `minipause`|
|candig|bool|If true, it allows `Beetle`'s [DoActionHold](Actions/DoActionHold.md) to happen if all other requirements are fufilled and this being true is required to use the `AntCompass` [item](../Enums%20and%20IDs/Items.md). It is set to true on the player's GroundDetector's OnTriggerStay when colliding with anything other than layer 13 (`NoDigGround`) or to false when colliding with anything on that layer. It is reset to false on the player's GroundDetector's OnTriggerExit|
|buttonhold|bool|If this is true, it means that the ability input is still being held after the end of a `flying` which occurs once `flyingcooldown` expires. This circumstance places the inputs processing in an inconsistent state so it will lock most inputs until the ability input is released where this field gets reset to false|
|submarine|bool|Tells if the player is currently in a submarine which affects many logic in the rest of the game including in PlayerControl such as entity rendering|
|tattling|bool|If true, it means that the player initiated the [SetText](../SetText/SetText.md) call to obtain help on `npc[0]` or the current map. It is set to false on LateUpdate when the [message](../SetText/Notable%20states.md#message) locks gets released|
|trueflip|bool|A value that is assigned during Update's regular movement inputs processing. It is assigned to false for holding input 2 (left) and to true for holding input 3 (right). The value is set to entity.`flip` when processing any tap action|
|boulderbreak|float|The amount of frames where during DashBehavior, even if entity.`hitwall` gets true or we enter a `pause`, the dash won't be stopped. This is only set to 5.0 when breaking a  [BreakableRock](../Entities/NPCControl/ObjectTypes/BreakableRock.md#breakrock) which prevents the wall detector from ending the dash as the rock that was in the way was just destroyed. The cooldown is updated on every LateUpdate cycle that's not in a `pause` or `minipause`|

## Respawn tracking

|Name|Type|Description|
|---:|----|-----------|
|lastloadzone|Vector3|The position to warp each `playerdata` (after adding `MainCamera`'s forward vector * 0.1 * `playerdata` index to prevent z fighting) during a Hazard's HazardAction when called for the 4th time while `movecd`  is 10.0 or above. This is only set at the end of a [TransferMap](../MapControl/Map%20loading.md#transfermap) and in some events to the player's position|
|lastpos|Vector3|A position to respawn the player under certain conditions such as a Hazard's HazardAction, getting below the map's [ylimit](../MapControl/Map%20entities.md#ylimit) in the y position, a [TempPlatform](../Entities/NPCControl/ObjectTypes/TempPlatform.md) that calls RespawnPlayer or a failsafe during [CheckItem](../Entities/NPCControl/ObjectTypes/Item.md#checkitem) when staying in the air for 300.0 frames or more|

## Collision tracking

|Name|Type|Description|
|---:|----|-----------|
|conveyor|StaticModelAnim|The StaticModelAnim of the GameObject with a `Conveyor` that was last collided with via OnTriggerStay, set to null on OnTriggerExit. This not being null will move the player by adding the StaticModelAnim's `conveyor` * `framestep` to the position on OnTriggerStay when colliding with the same StaticModelAnim as this field. This not being null also prevents the usage of the `AntCompass` key [item](../Enums%20and%20IDs/Items.md) This has `HideInInspector`, but since it's never assigned from prefab, it doesn't do anyting|
|standingon|Collider|The last collider the player's GroundDetector detected via the GroundDetector's OnTriggerStay, set to null on OnTriggerExit. It is used to see if additional logic needs to be done when standing on top of an active [NPCControl](../Entities/NPCControl/NPCControl.md)'s entity.`icecube` before calling [BreakIce](../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice). It is reset to null on the player's GroundDetector's OnTriggerExit|

## Coroutines

|Name|Type|Description|
|---:|----|-----------|
|jumproutine|Coroutine|A way for the game to store a JumpTo coroutine as this field is set to null when it is done so the game can yield on this field being null to wait that the coroutine is done|

## UNUSED

|Name|Type|Description|
|---:|----|-----------|
|flyheight|float|This is UNUSED|
|frozencube|Transform|This is UNUSED|
|ceiling|bool|This is practically UNUSED because it only has an effect when it is set to true by GroundDetector, but it requires `ceilingdetector` to be true and that field is also UNUSED. It would have indicated that the ceiling's GroundDetector collided with something through OnTriggerStay and it would have prevented any `flying` or jumping, but it would also override any other logic done in OnTriggerStay and OnTriggerExit to ignore the regular colliding logic|
|setfolloweronground|bool|This is UNUSED because it is never set to true and it would only have an effect if it was set to true. If it had been true, it would have cycled the follow chain such that `playerdata[1]` follows `playerdata[0]` and `playerdata[2]` follows `playerdata[1]` followed by the field being set to false. This follow chain cycling which would have caused unexpected behaviors, but it is unused under normal gameplay|

## Private
The following fields are all private so they are for internal trackings only:

|Name|Type|Description|
|---:|----|-----------|
|dashtarget|Vector3|The normalized version of the heading direction during `dashing` as compouted by DashBehavior during inputs processing. It is used to compute `dashdelta` which is the real movement vector used that combines the current direction with the new one and does a lerp between the 2 to give slightly delayed turns|
|digicon|SpriteRenderer[]|Should always contain 2 elements where both elements represents the UI icons to display when `digging` (first is the red arrow, second is `Beetle`'s icon). Both elements are initialised on Start|
|tunnelpart|GameObject|A `Digging` particle used during `Beetle`'s hold action (dig). On CancelAction, this object is moved offscreen then destroyed in 1.0 seconds|
|diggingpart|GameObject|Either a `DirtFlying` particle when using `Beetle`'s hold action (dig) or assigned to `smoke` when using `Beetle`s double tap action (dash). In the case of a dig, on the second DoActionHold call or CancelAction, this object is moved offscreen then destroyed in 1.0 seconds|
|bubbleshield|GameObject|An instance of `Prefabs/Objects/BubbleShield` childed to the entity.`rotater` with a local position of (0.5, 1.25, 0.0) with an initial scale of Vector3.zero so it's not seen initially. All of this is initialised on Start and the visibility is done by adjusting its scale towards (3.0, 3.0, 2.0) or Vector3.zero on LateUpdate depending on the `shield` field|
|tbox|BoxCollider|The BoxCollider of a GameObject named `cut` that is used during `Beetle`'s DoActionTap. Depending on if [flag](../Flags%20arrays/flags.md#flags) 39 is true or not (obtained upgraded dash), the tag of `cut` will be `BeetleHorn` or `BeetleDash` with the latter being the upgraded version. It is destroyed on StopDash and [CancelAction](../SetText/Individual%20commands/Cancelaction.md)|
|smoke|ParticleSystem|An instance of `Prefabs/Particles/WalkDust` used during a dash. It is moved offscreen and destroyed in 1.5 seconds in StopDash|
|keepdig|float|The amount of frames left to indicate PlayerControl that they should continue `digging` if one was initiated so it prevents to undig even if the ability input is released. This is set to 5.0 when in a [`KeepDig`](../MapControl/DigWall.md#keepdig) tagged collider (constantly renewed from OnTriggerStay) and set to 0.0 when leaving the collider|
|idletime|float|The amount of frames in a row since `delta` had a value of Vector3.zero updated on Update. The value won't go higher when it reaches 1000.0 or above and it only has an effect once it gets above 250.0 where LateUpdate may reveal the main HUD and the money HUD|
|lastactionid|int|This is set to 0 at the start of DoActionHold and DoActionTap (except if `Beetle` is the leader while not being in a `submarine` where it's set to 1). This is only used in ActionCooldown to set `actioncooldown` to 5.0 frames instead of 17.0 frames if the value is 1 (meaning that `Beetle` was doing their tap action)|
|actionroutine|Coroutine|The DoActionTap coroutine if one is in progress, null otherwise|
|footstep|float|The cooldown in frames used during [Movement](Movement.md) that when expired, the `Footstep` sound will play. It is set to 7.5 everytime the sound is played or to 0.0 when not moving. The cooldown is updated by the Movement method|
|tempcamoffset|Vector3?|This is practically UNUSED because while its value would be used to set instance.`camoffset` on [CancelAction](Actions/CancelAction.md) if the value isn't null, the value is never set to any other value than null so it stays at null under normal gameplay and doesn't do anything|
|ceildetect|GroundDetector|This is practically UNUSED because while it is the GroundDetector of a new `Prefabs/GroundDetector` GameObject created in the Ceiling method, that method is UNUSED under normal gameplay|
|startheight|float?|This is the y position modifier used during a `Bee`'s [DoActionHold](Actions/DoActionHold.md) (fly). It is null when the y position shouldn't be changed. When it's not null and the y position hasn't reached this field's value + 1.0, the y position is set to a lerp from the existing one to this field's value + 1.0 with a factor of 0.05. On every DoActionHold, the value gets set to the current y position so it's effectively a progressive lerp where the y position through this value keeps getting higher as the fly happens|
|respawncount|float|A frame counter that counts the amount of frames that the player was colliding with a `Respawn` tagged collider that isn't a [SetPlayerRespawn](../Entities/NPCControl/ObjectTypes/SetPlayerRespawn.md) (or it is one, but its `vectordata[0]` magnitude doesn't exceed 0.1) and was `onground`. Once this counter reaches above 15.0, `lastpos` is set to the current player position and this counter resets to 0.0. It is also reset to 0.0 when leaving the collider. More information can be found in the [Respawn](../MapControl/Respawn.md) documentation|
