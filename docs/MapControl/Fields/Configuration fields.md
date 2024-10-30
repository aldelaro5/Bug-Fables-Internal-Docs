# MapControl configuration fields
These fields are meant to be initialised from a prefab's MapControl. The MapControl is meant to be attached to the root object and it allows to assign values to most of the public fields in the Unity Editor.

Most fields are meant to be initialised this way (some may default to a value if they are optional). These fields are called configuration fields because they configure the game's behavior while on this map.

They are categorised by the specific documentation page describing the systems they are involved in.

## [Identification](../Map%20identification.md)
These fields are the bare minimum a map needs and can be considered the only required fields. They allow the game to identify the map and its logical group it belongs in.

|Name|Type|Description|Default|
|---:|----|----------|-------|
|mapid|MainManager.Maps|The unique identifier of this map. Every map should have a different value set from the others|`TestRoom` (must be assigned to a unique value)|
|areaid|MainManager.Areas|The area this map belongs to. On Start, if this field is different than instance.areaid, UpdateArea is called|`BugariaOutskirts` (should be assigned to the logically accurate value)|

## [Camera system](../Camera%20system.md)
These fields allows the configure the [main camera system](../../General%20systems/Camera%20system.md) while on this map.

|Name|Type|Description|Default|
|---:|----|----------|-------|
|camoffset|Vector3|The initial instance.`camoffset` of the camera while on this map and the value set when ResetCamera is called while on this map|MainManager.`defaultcamoffset` if the magnitude is 0.2 or lower|
|camangle|Vector3|The initial instance.`camangleoffset` of the camera while on this map and the value set when ResetCamera is called while on this map|MainManager.`defaultcamangle` if the magnitude is 0.2 or lower|
|camlimitneg|Vector3|The lower limit of the camera's position for this map expressed in each axises|(-999.0, 0.0, -999.0)|
|camlimitpos|Vector3|The upper limit of the camera's position for this map expressed in each axises|(999.0, 999.0, 999.0)|
|rotatecam|bool|If true, it changes the camera movement to look at the `actualcenter` when RefreshCamera is called which overrides the normal movement logic|false|
|centralpoint|Vector3|The initial value of `actualcenter` which is the center point of the radius limit. If `tieYtoplayer` is false or `camtarget` is null, this is effectively equivalent to `actualcenter`. Otherwise, the x and z components of `actualcenter` are equivalent to this field, but the y component gets overriden to the `camtarget` y position. Additionally, this field is the `center` of a Wind if its `owncenter` is false on its Start|Vector3.zero|
|tieYtoplayer|bool|When true, it causes the y component of `actualcenter` to be overriden in FixedUpdate to the `camtarget` y poisition if it's not null which effectively lets the camera follow its target in the y axis. If this field and `rotatecam` are true on RefreshCamera, the camera x angle is always set to 0.0 which fixes the x rotation in place|false|
|tetherdistance|float|If above 0, this becomes the max radius from `actualcenter` the camera can be positioned at during RefreshCamera via the radius limit|-1.0, but it is overriden if `tetherYLerp` x component is above 0.0|
|tetherYLerp|Vector3|Only applies if `tieYtoplayer` is true and `camtarget` isn't null. This field contains the 3 parameters to a lerp operation to do on FixedUpdate to update `tetherdistance` if the x component is above 0. The lerp will be done from the x component to the y component with a factor of `camtarget` y position / the z component. NOTE: This feature is complicated, see the [tetherYLerp details](../Camera%20system.md#more-details-on-tetherylerp) section in the camera system page for more details|Vector3.zero|
|roundways|Transform[]|Only the first element matters. If the first element exists, `rotatecam` is true and we aren't in an [inside](../Insides.md), the corresponding Transform will be positioned relative to the camera's forward vector on FixedUpdate in x and z (the y component is always 5.0). The position is multiplied by `lightoffset` before being set. NOTE: This is very specific to the `AntTunnels` [map](../../Enums%20and%20IDs/Maps.md)|Empty array|
|lightoffset|float|If `roundways[0]` exists, `rotatecam` is true and we aren't in an inside, this field will be a multiplier applied to the position to set `roundways[0]` in FixedUpdate. NOTE: This is very specific to the `AntTunnels` [map](../../Enums%20and%20IDs/Maps.md)|5.0|

## Graphics

|Name|Type|Description|Default|
|---:|----|----------|-------|
|fogend|float|The starting value of RenderSettings.fogEndDistance|300.0|
|fogcolor|Color|The starting value of RenderSettings.fogColor|Pure white|
|screeneffect|ScreenEffects|If this is `SunRaysTopRight`, inside SetScreenEffect, a new `Prefabs/Particles/SunRay` is instantiated childed to the `GUICamera` with a local position of (9.0, 7.0, 10.0) without rotations|`None`|
|skyboxmat|Material|The skybox material to use for this map when this field isn't null (also will have its shader `_Tint` set to pure gray when not in an inside). If this field is null, skybox is set to null (so no skybox), but MainManager.`MainCamera`.backgroundColor will be set to pure black|null|
|overrideskybox|bool|If true, it prevents FixedUpdate to update the `_Rotation` shader property of the skybox. It also prevents any material's color to be updated in FixedUpdate when entering or exiting an inside. NOTE: It is NOT recommended to set this to true because it has unintended side effects with [insides](../Insides.md)|false|
|globallight|Color|The starting value of RenderSettings.ambientLight|Pure gray|
|windspeed|float|Configures the range of values possible for the `_ShakeDisplacement` shader property of a material using the MainManager.`windshader`. This shader is only used during the SetUp of a [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md) when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field. NOTE: Because this property is unused in that shader, this value effectively does nothing. The intended property was likely `_ShakeWindspeed` which would have configured the speed of the shaking, but it is not configurable under normal gameplay|0.2|
|windintensity|float|Configures the range of values possible for the `_ShakeBending` shader property of a material using the MainManager.`windshader` (this property controls the amount of warping to apply to the object as a result of the wind). This shader is only used during the SetUp of a [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md) when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field|0.2|
|faderchange|bool|If true, all Fader will fade all their Renderer instead of culling them completely. Also, on Start, `faders` and `fss` gets their initialised on the first LateUpdate which are used in LateUpdate to reset all the sharedMaterials, shadowCastingMode and sharedMaterials's renderQueue of all disabled Fader's renders that were initially enabled|false|
|skycolor|Color|If a Fader is set to fade its GameObject's materials (forced to be the case if `faderchange` is true), this is the color tint to set the materials's colors to in RGB (the A component is ignored since Fader controls it)|Pure white|

## Battles

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|battlemap|MainManager.BattleMaps|The default BattleMaps to use when BattleControl.[StartBattle](../../Battle%20system/StartBattle.md) is called on this map with a stageid of -1. This is also the default BattleMaps used for a CardBattle started with a mapid of -1|`Grasslands1` (should be assigned to the logically accurate value)|
|battleleaftype|BattleLeafType|The type of transition to use on this map when [StartBattle](../../Battle%20system/StartBattle.md) is called which differs visually. If the value is `Bee`, the `BattleStart3` sound will play instead of `BattleStart0`|`Common`|
|expmulti|float|An EXP amount multiplier that affects the EXP scaling of MainManager.[GetEXP](../../TextAsset%20Data/Enemies%20data.md#exp-logic) while called on this map. This should NEVER be negative or unexpected behaviors can occur|1.0|
|battleleafcolor|Color|The default color of the visual transition of `battleleaftype` when [StartBattle](../../Battle%20system/StartBattle.md) is called and adv isn't 3 (not an enemy party advantage)|Pure green|
|nobattlemusic|bool|If this is true, it will prevent [StartBattle](../../Battle%20system/StartBattle.md) from changing the music. This is a way for a map to keep the music the game was already playing during the battle without interruption. NOTE: This field is involved in the [music playback issue](../../General%20systems/Music%20playback.md#issue-with-musicresume)|false|

## Music

|Name|Type|Description|Default|
|---:|----|----------|-------|
|music|AudioClip[]|The list of AudioClip this map has access to play as background music. If the array is empty, the current music will fade int silence and stop playing|Empty array|
|musicid|int|The default `music` index to use as the background music, but it can be overriden if `musicflags` isn't empty. If `musicflags` isn't empty, it defaults to -1 before having the value be determined by `musicflags` (or stays at -1 if no element is selected). Regardless of `musicflags`, if the value ends up at -1, the current music will fade into silence before it stops playing|0 (-1 if `musicflags` isn't empty)|
|keepmusic|bool|When this is true and instance.`inevent` is false, [Music](../Init%20methods/Music.md) won't be called by Start which means the current music playing won't change and `musicid`'s value won't be changed. Essentially, it prevents the music that was playing before loading this map from changing when loading this map|false|
|musicflags|Vector2Int[]|A list of directives to follow regardinig the assignement of `musicid` in the [Music](../Init%20methods/Music.md) method. When not empty, `musicid` defaults to -1 followed by each element being processed in reverse order. The first one whose x component is either -1 or a [flags](../../Flags%20arrays/flags.md) slot that is true will be selected. When selected, `musicid` gets set to the y component's value and it will be used as the current music. If no suitable element is selected, `musicid` stays at -1 and the current music will fade into silence before it stops playing|Empty array|

## Insides

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|insides|GameObject[]|The list of GameObject in this map that is considered an interior part and will get special treatment during rendering and include special transition managed by [DoorSameMap](../../Entities/NPCControl/ObjectTypes/DoorSameMap.md) map entities|Empty array|
|insidetypes|InsideType[]|This is meant to be a matching array for `insides` and each matching element corresponds to the sound that will play when entering or exiting the inside. This field's string value is the prefix to the sound's name: when entering the inside, the name of the sound that will play ends with `DoorEnter` and when exiting it, it will end with `DoorExit`. Concatenating this field's string value as the prefix gives the full sound name to play when entering or exiting the inside|Empty array, but if the length doesn't match `insides` on Start, it is reset to be an array with the same length, but with all values left to `Stretch`|
|tieinsidedoorentities|bool|If this is true when entering an inside, every [DoorSameMap](../../Entities/NPCControl/ObjectTypes/DoorSameMap.md) in the map has their `vectordata[4]` and `vectordata[5]` set to the current `camoffset` / `camangleoffset` which means no matter how the inside is exited, the camera values will be restored consistently to the values when it was entered|false|
|hideinsides|bool|If this is true, all `insides` that doesn't match the current one will be disabled. This will also disable any entity with `hideinside` set to true whose `insideid` doesn't match the current one. This is only used in the `AntTunnels` [map](../../Enums%20and%20IDs/Maps.md) to hide the break room part of the map|false|
|setinsidecenter|bool|If this is true, whenener RefreshInsides is called, instance.`camtarget` will be set to the current inside's transform after entering one and set back to MainManager.`player`'s transform when exiting one|false|
|fadingspeed|float|The fraction of TieFramerate to use in FixedUpdate when updating `fadeammount` when entering or exiting an inside|0.2|

## SetText

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|useglobalcommand|bool|Tells if the global commands system is enabled for this map. Since this system isn't complete in its implementation, this should normally not be true. See the section below for details|false|
|tattleid|int|The SetText [dialogue line id](../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) that will be used when pressing the Help input on this map away from an NPC|0|
|englishbreakfix|bool|If true, it changes OrganizeLines when instance.`languageid` is `English` for all SetText calls made in dialogue mode such that the fixed logic applies to fix the [whole word width line skip](../../SetText/Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-words-width-after-the-first-line). This is an optional fix the map needs to opt in because it can have unintended side effects as the entire English script was written with the expectation that this issue was always present|false|

## Followers

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|canfollowID|int[]|The [animIDs](../../Enums%20and%20IDs/AnimIDs.md) that are allowed to follow the player party on this map when present in instance.`extrafollowers`. This restriction is ignored if the GameObject's name is `0` (which is normally only the case for the `TestRoom` [map](../../Enums%20and%20IDs/Maps.md))|Empty array|
|followerylimit|float|The maximum following distance (absolutely value) in y allowed for any entity following another. If the y distance between the entity and its followee gets higher than this value and we aren't in a `minipause`, the [DoFollow](../../Entities/EntityControl/Notable%20methods/Follow.md#dofollow) of the entity will teleport them to their followee instantly. This should NEVER be negative because the distance is meant to be expressed as an absolute value|20.0|
|closemove|bool|This field if true allows the map to force the CloseMove entity follow logic of the player party. NOTE: The influence of this fiels is extremely complex and inconsistent, primarily during the `BanditHideout` stealth section, more details can be found in the [follower system](../Follower%20system.md) documentation|false|

## Entities

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|ylimit|float|A lower bound limit of the y position of entities in the map. If a player or map entity's y position gets lower than this value, they will respawn. The value is overriden to -150.0 if there's a Hazard present with a `type` of `Hole` due to its Start logic|-50.0|
|icemap|bool|If this is true, the entire map is treated as an ice map meaning every entity's Start marks their `inice` to true which has several ice related changes|false|
|limitbehavior|bool|If this is true, it restricts more what is considered an active NPCControl. For Update, it changes the maximum z `campos` allowed before becoming inactive from 25.0 to 15.0. For LateUpdate, it mandates that the entity is `incamera` for `inrange` updates to happen|false|
|keepobjectsactive|bool|If this is true, LateUpdate won't update the enablement of too far away NPCControl and disable their `emoticon` every 2 frames (after `alivetime` reached 0.0)|false|


## Miscellaneous

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|mainmesh|Transform|The transform containing the "main" mesh which is a major reference points for the map. It determines the default value of `mainrender` (the Renderer of `mainmesh`) and the value of `render` (all the Renderer childed under `mainmesh`)|The first child of MapControl if null|
|discoveryids|int[]|The list of [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) ids that this map contains. The ones whose flags are still false becomes detectable by the `Detector` [medal](../../Enums%20and%20IDs/Medal.md) when equipped (except if the `mapid` is the `TermiteIndustrial` [map](../../Enums%20and%20IDs/Maps.md) while the player z position is less than 20.0 where no discoveries can be detected)|Empty array|
|readdatafromothermap|MainManager.Maps|From prefab|When set to any value other than `TestRoom`, it will cause Start to read the `dialogues`, `commandlines` and `entities` TextAsset data from that map instead of the ones from `mapid`|`TestRoom`|
|preloadobjs|GameObject[]|A dummy list of GameObject that allows to preemptively load prefabs so they can be cached if they get loaded again. This only holds references, the actual GameObjects aren't used from this field|Empty array|
|alivetime|float|The amount of frames (excluding the first one) before LateUpdate is allowed to restore the disabled, but initially enabled [Fader](../Graphics%20configuration.md#fader-control) and also to update the enablement of [DoorOtherMap](../../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) and [NPC](../../Entities/NPCControl/NPC.md) (including their `emoticon`) |20.0|
|cantcompass|bool|If this is true, the `AntCompass` [key item](../../Enums%20and%20IDs/Items.md) cannot be used on this map|false|
|autoevent|Vector2[]|A list of [events](../../Enums%20and%20IDs/Events.md) in the y component to automatically start the moment the [flag](../../Flags%20arrays/flags.md) slot of the x component is false. This is checked on each LateUpdate after the first one and when the event starts, the flag slot of the x component becomes true so it doesn't start the event again|Empty array|

## Unused

|Name|Type|Description|
|---:|----|----------|
|insidecamspeed|float|From prefab|UNUSED|1.0|
|transferspeed|float|From prefab|UNUSED|0.2|
|baseoutline|float|From prefab|UNUSED|20.0|

