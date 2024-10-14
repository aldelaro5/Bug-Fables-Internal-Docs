# MapControl fields

## Configuration from prefab

### Identification

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|mapid|MainManager.Maps|From prefab|The unique identifier of this map. Every map should have a different value set from the others|`TestRoom` (must be assigned to a unique value)|
|areaid|MainManager.Areas|From prefab|The area this map belongs to. On Start, if this field is different than instance.areaid, UpdateArea is called|`BugariaOutskirts` (should be assigned to the logically accurate value)|

### Camera

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|camlimitneg|Vector3|From prefab|The lower limit of the camera's position for this map expressed in each axises|(-999.0, 0.0, -999.0)|
|camlimitpos|Vector3|From prefab|The upper limit of the camera's position for this map expressed in each axises|(999.0, 999.0, 999.0)|
|camangle|Vector3|From prefab|The initial instance.`camangleoffset` of the camera while on this map and the value set when ResetCamera is called while on this map|MainManager.`defaultcamangle` if the magnitude is 0.2 or lower|
|camoffset|Vector3|From prefab|The initial instance.`camoffset` of the camera while on this map and the value set when ResetCamera is called while on this map|MainManager.`defaultcamoffset` if the magnitude is 0.2 or lower|
|rotatecam|bool|From prefab|If true, it changes the camera movement to look at the `actualcenter` when RefreshCamera is called which overrides the normal movement logic|false|
|centralpoint|Vector3|From prefab|The initial value of `actualcenter`. If `tieYtoplayer` is false or `camtarget` is null, this is effectively equivalent to `actualcenter`. Otherwise (`tieYtoplayer` is true and `camtarget` isn't null), the x and x components of `actualcenter` are equivalent to this field, but the y component gets overriden to the `camtarget` y position. Additionally, this field is the `center` of a Wind if its `owncenter` is false on its Start|Vector3.zero|
|tieYtoplayer|bool|From prefab|When true, it causes the y component of `actualcenter` to be overriden in FixedUpdate to the `camtarget` y poisition if it's not null which effectively lets the camera follow its target in the y axis. If this field and `rotatecam` are true, the camera x angle is always set to 0.0 which fixes the x rotation in place|false|
|roundways|Transform[]|From prefab|Only the first element matters. If the first element exists, `rotatecam` is true and we aren't in an inside, the corresponding Transform will be positioned relative to the camera's forward vector on FixedUpdate in x and z (the y component is always 5.0). TODO: explain intuitively how this works The position is multiplied by `lightoffset` before being set|Empty array|
|lightoffset|float|From prefab|If `roundways[0]` exists, `rotatecam` is true and we aren't in an inside, this field will be a multiplier applied to the position to set `roundways[0]` in FixedUpdate|5.0|
|tetherYLerp|Vector3|From prefab|Only applies if `tieYtoplayer` is true and `camtarget` isn't null. This field contains the 3 parameters to a lerp operation to do on FixedUpdate to update `tetherdistance` if the x component is above 0. The lerp will be done from the x component to the y component with a factor of `camtarget` y position / the z component TODO: what does this do intuitively?|Vector3.zero|
|tetherdistance|float|From prefab|If above 0 (meaning `tetherYLerp` took effect), this becomes the max radius from `actualcenter` the camera can be positioned at during RefreshCamera TODO: recheck why this restriction is needed on top of cam limits which would already do this|-1.0|

### Graphics

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|screeneffect|ScreenEffects|From prefab|If this is `SunRaysTopRight`, inside SetScreenEffect, a new `Prefabs/Particles/SunRay` is instantiated childed to the `GUICamera` with a local position of (9.0, 7.0, 10.0) without rotations|`None`|
|skyboxmat|Material|From prefab|The skybox material to use for this map when this field isn't null (also will have its shader `_Tint` set to pure gray when not in an inside). If this field is null, skybox is set to null (so no skybox ), but MainManager.`MainCamera`.backgroundColor will be set to pure black|null|
|fogend|float|From prefab|The starting value of RenderSettings.fogEndDistance|300.0|
|fogcolor|Color|From prefab|The starting value of RenderSettings.fogColor|Pure white|
|globallight|Color|From prefab|The starting value of RenderSettings.ambientLight|Pure gray|
|windspeed|float|From prefab|Configures the range of values possible for the `_ShakeDisplacement` shader property of a material using the MainManager.`windshader`. This shader is only used during the SetUp of a BeetleGrass when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field|0.2|
|windintensity|float|From prefab|Configures the range of values possible for the `_ShakeBending` shader property of a material using the MainManager.`windshader`. This shader is only used during the SetUp of a BeetleGrass when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field|0.2|
|faderchange|bool|From prefab|If true, all Fader will fade all their Renderer instead of culling them completely. Also, on Start, `faders` and `fss` gets their initialised on the first LateUpdate which are used in LateUpdate to reset all the sharedMaterials, shadowCastingMode and sharedMaterials's renderQueue of all disabled Fader's renders that were initially enabled TODO: figure out what exactly this logic does, it's very off and compares ref between materials and sharedMaterials which seems wrong, was changed in 1.1.0 beta rev13|false|
|overrideskybox|bool|From prefab|If true, it prevents FixedUpdate to update the `_Rotation` shader property of the skybox. It also prevents any material's color to be updated in FixedUpdate when entering or exiting an inside|false|
|skycolor|Color|From prefab|If a Fader is set to fade its GameObject's materials (forced to be the case if `faderchange` is true), this is the color tint to set the materials's colors to in RGB (the A component is ignored since Fader controls it)|Pure white|

### Battles

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|battlemap|MainManager.BattleMaps|From prefab|The default BattleMaps to use when BattleControl.StartBattle is called on this map with a stageid of -1. This is also the default BattleMaps used for a CardBattle started with a mapid of -1|`Grasslands1` (should be assigned to the logically accurate value)|
|battleleaftype|BattleLeafType|From prefab|The type of transition to use on this map when StartBattle is called which differs visually. If the value is `Bee`, the `BattleStart3` sound will play instead of `BattleStart0`|`Common`|
|expmulti|float|From prefab|An EXP amount multiplier that affects the EXP scaling of GetEXP while called on this map. This should NEVER be negative or unexpected behaviors can occur|1.0|
|battleleafcolor|Color|From prefab|The default color of the visual transition of `battleleaftype` when StartBattle is called and adv isn't 3 (not an enemy party advantage)|Pure green|
|nobattlemusic|bool|From prefab|If this is true, it will prevent StartBattle from changing the music. This is a way for a map to keep the music the game was already playing during the battle without interruption|false|


### Music

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|musicid|int|From prefab (from Start if `musicflags` isn't empty)|The default `music` index to use as the background music if `musicflags` is empty. If `musicflags` isn't empty, it defaults to -1 before having the value be determined by `musicflags` (or stays at -1 if no element is selected). Regardless of `musicflags`, if the value ends up at -1, the current music will fade into silence before it stops playing|0 (-1 if `musicflags` isn't empty)|
|music|AudioClip[]|From prefab|The list of AudioClip this map has access to play as background music. The one at the index `musicid` is the default one (if `musicflags` is empty), but it can be changed to any other AudioClip from this list. If the array is empty, the current music will fade int silence and stop playing|Empty array|
|keepmusic|bool|From prefab|When this is true and instance.`inevent` is false, Music won't be called which means the current music playing won't change and `musicid`'s value won't be changed. Essentially, it prevents the music that was playing before loading this map from changing when loading this map|false|
|musicflags|Vector2Int[]|From prefab|A list of directives to follow regardinig the assignement of `musicid` in the Music method. When not empty, `musicid` defaults to -1 followed by each element being processed in reverse order. The first one whose x component is either -1 or a flags slot that is true will be selected. When selected, `musicid` gets set to the y component's value and it will be used as the current music. If no suitable element is selected, `musicid` stays at -1 and the current music will fade into silence before it stops playing|Empty array|

### Insides

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|hideinsides|bool|From prefab|If this is true, all `insides` that doesn't match the current one will be disabled. This will also disable any entity with `hideinside` set to true whose `insideid` doesn't match the current one. This is only used in the `AntTunnels` [map](../../../Enums%20and%20IDs/Maps.md) to hide the break room part of the map|false|
|insides|GameObject[]|From prefab|The list of GameObject in this map that is considered an interior part and will get special treatment during rendering and include special transition managed by DoorSameMap map entities. TODO: check more info|Empty array|
|insidetypes|InsideType[]|From prefab (may be resized on Start)|This is meant to be a matching array for `insides` and each matching element corresponds to the sound that will play when entering or exiting the inside. This field's string value is the prefix to the sound's name: when entering the inside, the name of the sound that will play ends with `DoorEnter` and when exiting it, it will end with `DoorExit`. Concatenating this field's string value as the prefix gives the full sound name to play when entering or exiting the inside|Empty array, but if the length doesn't match `insides` on Start, it is reset to be an array with the same length, but with all values left to `Stretch`|
|setinsidecenter|bool|From prefab|If this is true, whenener RefreshInsides is called, instance.`camtarget` will be set to the current inside's transform after entering one and set back to MainManager.`player`'s transform when exiting one|false|
|tieinsidedoorentities|bool|From prefab|If this is true, when entering any inside via MoveInside, all DoorSameMap except the caller will have their `vectordata[4]` and `vectordata[5]` set to the ones from the caller which were just set to the instance.`camoffset` and instance.`camangleoffset` respectively. This effectively allows an inside to have multiple DoorSameMap leading to it, but still maintain camera fields consistency when exiting it from a different DoorSameMap than the one that was used to enter the inside|false|
|fadingspeed|float|From prefab|The fraction of TieFramerate to use in FixedUpdate when updating `fadeammount` when entering or exiting an inside|0.2|

### SetText

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|useglobalcommand|bool|From prefab|Tells if the global commands system is enabled for this map. Since this system isn't complete in its implementation, this should normally not be true|false|
|tattleid|int|From prefab|The SetText dialogue line id that will be used when pressing the Help input on this map away from an NPC|0|
|englishbreakfix|bool|From prefab|If true, it changes OrganizeLines when instance.`languageid` is `English` for all SetText calls made in dialogue mode such that the fixed logic applies to fix the issue where the first word width of a line isn't counted in the line length accumulator. This is an optional fix the map needs to opt in because it can have unintended side effects as the entire English script was written with the expectation that this issue was always present|false|

### Followers

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|canfollowID|int[]|From prefab|The animIDs that are allowed to follow the player party on this map when present in instance.`extrafollowers`. This restriction is ignored if the GameObject's name is `0` (which is normally only the case for the `TestRoom`)|Empty array|
|followerylimit|float|From prefab|The maximum following distance (absolutely value) in y allowed for any entity following another. If the y distance between the entity and its followee gets higher than this value and we aren't in a `minipause`, the DoFollow of the entity will teleport them to their followee instantly. This should NEVER be negative because the distance is meant to be expressed as an absolute value|20.0|
|closemove|bool|From prefab|This field if true allows the map to force the CloseMove entity follow logic of the player party. However, this field being true isn't alone to force the CloseMove logic: flag 401 needs to be true (during a stealth section) alongside the current map having this field set to true. NOTE: In practice, including some AreaSpecific logic, both are true when needed with the exception of the Bandit Hideout stealth section where flag 401 is true until exiting the first map of the stealth section (right after the event that starts it)|false|

### Entities

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|ylimit|float|From prefab (may be overriden by Hazard's Start)|A lower bound limit of the y position of entities in the map. If a player or map entity's y position gets lower than this value, they will respawn. The value is overriden to -150.0 if there's a Hazard present with a `type` of `Hole` due to its Start logic|-50.0|
|icemap|bool|From prefab|If this is true, the entire map is treated as an ice map meaning every entity's Start marks their `inice` to true, Enemy NPCControl won't thaw out and PushRock configured as sliding ice cube are allowed to be moved|false|
|limitbehavior|bool|From prefab|If this is true, it restricts more what is considered an active NPCControl. For Update, it changes the maximum z `campos` allowed before becoming inactive from 25.0 to 15.0. For LateUpdate, it mandates that the entity is `incamera` for `inrange` updates to happen|false|
|keepobjectsactive|bool|From prefab|If this is true, LateUpdate won't update the enablement of too far away NPC and disable their `emoticon` every 2 frames (after `alivetime` reached 0.0)|false|


### Miscellaneous

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|mainmesh|Transform|From prefab|The transform containing the "main" mesh which can be used as a reference point to other objects of the map|The first child of MapControl if null|
|discoveryids|int[]|From prefab|The list of discovery ids that this map contains. The ones whose flags are still false becomes detectable by the `Detector` medal when equipped (except if the `mapid` is `TermiteIndustrial` while the player z position is less than 20.0 where no discoveries can be detected)|Empty array|
|readdatafromothermap|MainManager.Maps|From prefab|When set to any value other than `TestRoom`, it will cause Start to read the `dialogues`, `commandlines` and `entities` TextAsset data from that map instead of the ones from `mapid`|`TestRoom`|
|preloadobjs|GameObject[]|From prefab|A dummy list of GameObject that allows to preemptively load prefabs so they can be cached if they get loaded again. This only holds references, the actual GameObjects aren't used from this field|Empty array|
|alivetime|float|From prefab|The amount of frames (excluding the first one) before LateUpdate is allowed to update the faders (???) and some specific NPCControl every 2 frames (DoorOtherMap and NPC)|20.0|
|cantcompass|bool|From prefab|If this is true, the `AntCompass` key item cannot be used on this map|false|
|autoevent|Vector2[]|From prefab|A list of events in the y component to automatically start the moment the flag slot of the x component is false. This is checked on each LateUpdate after the first one and when the event starts, the flag slot of the x component becomes true so it doesn't start the event again|Empty array|

### Unused

|Name|Type|Description|
|---:|----|----------|
|insidecamspeed|float|From prefab|UNUSED|1.0|
|transferspeed|float|From prefab|UNUSED|0.2|
|baseoutline|float|From prefab|UNUSED|20.0|


## Public tracking fields

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
|currentline|int|Not initialised|TODO: explain the command stuff|-1 (meant to stay with that value until needed)|
|entityonly|Collider[]|From SetPlayerColliders which is invoked 0.2 seconds after the first LateUpdate|A list containing the first Collider of every GameObject with tag `EntityOnly`. Each of these colliders has their staticFriction and dynamicFriction set to 0.0 and are ignored by every `playerdata` and `tewpfollowers` entities TODO: more ???|Empty array, but this is always set to the needed value|
|stencilid|int|From LateUpdate after the first one|The map entity id of the first StencilSwitch who isn't `iskill` and its `hit` is true. If none are present, the value is -1|-1 (this is meant to constantly get updated)|
|lastwater|Hazards|From Hazards's Start if needed (has HideInInspector attribute)|This is set to the last Hazards of type `Water` or `Honey` (set on the Hazards's Start). If multiple Hazards of these types exists, this field is set to the last one whose Start was called|null|

## Private fields

|Name|Type|Description|
|---:|----|----------|
|tempcpos|Vector3|The value of `camlimitpos` when RemoveLimit was called with settemp. Used for restoring the value of `camlimitpos` in RestoreLimit with restoretemp|
|tempcneg|Vector3|The value of `camlimitneg` when RemoveLimit was called with settemp. Used for restoring the value of `camlimitneg` in RestoreLimit with restoretemp|
|tcpos|Vector3?|The value of `camlimitpos` when entering an inside. Used for restoring the value of `camlimitpos` in when exiting an inside TODO: recheck stuff|
|tcneg|Vector3?|The value of `camlimitneg` when entering an inside. Used for restoring the value of `camlimitneg` in when exiting an inside TODO: recheck stuff|
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
