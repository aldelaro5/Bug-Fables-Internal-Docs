# MapControl tracking fields

## Public

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|actualcenter|Vector3|From Start|The origin point of the map for the purpose of positioning the camera. If `rotatecam` is true, this is the point to look at. If it's false, this is the center of a circle with radius `tetherdistance` that the camera position must be in (this isn't enforced if `tetherdistance` isn't above 0.0 or in an inside)|Always set to `centerpoint` on Start|
|entities|EntityControl[]|From Start (CreateEntities)|The list of NPCControl of this map which is initialised in CreateEntities from the map entity data of this map|Empty list|
|dialogues|string[]|From Start|The list of SetText lines this map offers, read from the matching TextAsset data corresponding to the current instance.`languageid`|Empty list (only if the dialogue file couldn't be loaded)|
|mapflags|bool[]|From Start|A list of flags whose value only live throughout the lifetime of this map. Only contains 10 slots all initialised to false|A new array of 10 elements, all set to false|
|tempfollowers|List<EntityControl>|Always empty list initially|The current list of entities that is following the player party in the map including `chompy`|Always empty list initially|
|mainrender|Renderer|From Start (has HideInInspector attribute)|The first Renderer component in the children of `mainmesh`, UNUSED|Always set by Start when not null (which is always the case)|
|commandlines|string[]|From Start|The list of commands strings read from the matching TextAsset data if `useglobalcommand` is true|Empty array|
|hiddenitem|int?|From Start when needed|The value is used like a flag where a null value is false and true for anything not null. This value is only set to true when the `Detector` medal should beep if equipped on the next applicable LateUpdate (not in a `pause`, `minipause`, `inevent` or `message`). It is not meant to have the value come from the prefab as the value can be set to 100 in CheckDisc when verifying the `discoveryids` values. This can be set externally to force the medal to beep|null|
|chompy|EntityControl|From LateUpdate (`latestart`)|The entity corresponding to a special `tempfollowers` element with the `ChompyChan` animid. This follower is only added on the first LateUpdate when flag 402 is true (Chompy is with Team Snakemouth) and the player isn't in a `submarine`. If the follower is never added, this field remains null which the game can check for Chompy's presence. This value should NEVER be non null in the prefab or unexpected behaviors will occur|null|
|musicrangemain|int|From Start (CreateEntities)|The map entity id of the last MusicRange defined in this map. This MusicRange is the only one that is allowed to control the music's volume. If multiple are defined, this field's value will be the last occurrence's id|-1|
|samira|NPCControl|Not initialised|This field is meant for event 28 (talking to Samira) to assign the Samira's NPCControl to it. This allows this map to keep a reference so that on MoveInside, the game can make her stop playing her music playing animation if this field isn't null|null (meant to stay null until assigned)|
|entitysprite|List<Texture2D>|0.1 seconds after the first LateUpdate|A list of all the NPCControl's entities's `sprite`'s texture. This list is kept for faster loading performance when loading the textures again|Empty list (meant to be assigned soon after Start)|
|waterfloat|Hazards|Hazards's Start if called whode GameObject has a `WaterFloat` tag (has HideInInspector attribute)|This is a reference to the water Hazards which allows specific entities to move over it (as if they were floating over it) when allowed to. If multiple Hazards with a `WaterFloat` tag exists in the map, it is undefined which one will be selected as this value|null (meant to stay null unless a Hazards object exists under the map with a `WaterFloat` tag)|
|latestart|bool|From the first LateUpdate (has HideInInspector attribute)|A way to track whether or not the first LateUpdate happened. The first LateUpdate contains special initialisation logic outside of Start and this field tracks if this logic already happened or not|false (meant to be set to true on the first LateUpdate)|
|currentline|int|Not initialised|This tracks the SetText line to process when using the [GlobalCommand](../../SetText/Related%20Systems/GlobalCommand.md) system. It is expected to be set before SetText processes the text and reset to -1 when not processing anything or when the system isn't in use|-1 (meant to stay with that value until needed)|
|entityonly|Collider[]|From SetPlayerColliders which is invoked 0.2 seconds after the first LateUpdate|A list containing the first Collider of every GameObject with tag `EntityOnly`. More information can be found at the [EntityOnly](../EntityOnly.md) documentation|Empty array, but this is always set to the needed value|
|stencilid|int|From LateUpdate after the first one|The map entity id of the first StencilSwitch who isn't `iskill` and its `hit` is true. If none are present, the value is -1|-1 (this is meant to constantly get updated)|
|lastwater|Hazards|From Hazards's Start if needed (has HideInInspector attribute)|This is set to the last Hazards of type `Water` or `Honey` (set on the Hazards's Start). If multiple Hazards of these types exists, this field is set to the last one whose Start was called|null|

## Private

|Name|Type|Description|
|---:|----|----------|
|tempcpos|Vector3|The value of `camlimitpos` when RemoveLimit was called with settemp. Used for restoring the value of `camlimitpos` in RestoreLimit with restoretemp|
|tempcneg|Vector3|The value of `camlimitneg` when RemoveLimit was called with settemp. Used for restoring the value of `camlimitneg` in RestoreLimit with restoretemp|
|tcpos|Vector3?|The value of `camlimitpos` when entering an inside. Used for restoring the value of `camlimitpos` in when exiting an inside|
|tcneg|Vector3?|The value of `camlimitneg` when entering an inside. Used for restoring the value of `camlimitneg` in when exiting an inside|
|refreshmats|bool|This is to track if it should be needed to restore the sharedMaterials of all `render` to `ogmat`. This defaults to true and is set to true only when `fadeammount` is updating due to an inside transition. Once the fading updates are done, this value will indicate on the next FixedUpdate that the materials should be restored. When the materials are restored, this is set to false so no materials changes will happen until the next inside transition|
|windobjects|Renderer[]|Practically UNUSED because while it's used in RefreshWind, the method is unused and no one sets a value to this field. Since this is private, it can't be set from the prefab so this field doesn't influence anything|
|originallimitneg|Vector3|The initial value of `camlimitneg` on Start|
|originallimitpos|Vector3|The initial value of `camlimitpos` on Start|
|faders|Fader[]|If `faderchange` is true, this gets assigned to all Fader in the map on the first LateUpdate|
|fss|bool[]|If `faderchange` is true, this gets assigned to all Fader's enablement on the first LateUpdate|
|render|MeshRenderer[]|All the MeshRenderer in the children of `mainmesh`|
|fadeammount|float|An internal tracking value of the r, g and b that each `render` material colors should have updated to from FixedUpdate as the inside state changes. Starts at 1.0 which is fully colored|
|ogmat|Material[][]|All the materials arrays of every `render` elements|
|musicpreload|List<AudioClip>|A list of preloaded AudioClip gathered during CreateEntities by accumulating all the DoorSameMap's musics and preloading their AudioClip. This is a caching optimisation. Start always ensure it is not null|
|digwall|Collider[]|All the first Collider component of every GameObject with the `DigWall` tag|
|nocolorchange|bool|AreaSpecific may set this to true which prevents FixedUpdate from updating the `render` material color when entering or exiting an inside|
