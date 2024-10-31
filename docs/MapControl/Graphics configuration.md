# Map graphics configuration
There are many ways the map can customise the graphical rendering of the game through configurations.

It involves the following configuration fields:

|Name|Type|Description|Default|
|---:|----|----------|-------|
|fogend|float|The starting value of RenderSettings.fogEndDistance|300.0|
|fogcolor|Color|The starting value of RenderSettings.fogColor|Pure white|
|screeneffect|ScreenEffects|If this is `SunRaysTopRight`, inside SetScreenEffect, a new `Prefabs/Particles/SunRay` is instantiated childed to the `GUICamera` with a local position of (9.0, 7.0, 10.0) without rotations|`None`|
|skyboxmat|Material|The skybox material to use for this map when this field isn't null (also will have its shader `_Tint` set to pure gray when not in an inside). If this field is null, skybox is set to null (so no skybox), but MainManager.`MainCamera`.backgroundColor will be set to pure black|null|
|overrideskybox|bool|If true, it prevents FixedUpdate to update the `_Rotation` shader property of the skybox. It also prevents any material's color to be updated in FixedUpdate when entering or exiting an inside. NOTE: It is NOT recommended to set this to true because it has unintended side effects with [insides](Insides.md)|false|
|globallight|Color|The starting value of RenderSettings.ambientLight|Pure gray|
|windspeed|float|Configures the range of values possible for the `_ShakeDisplacement` shader property of a material using the MainManager.`windshader`. This shader is only used during the SetUp of a [BeetleGrass](../Entities/NPCControl/ObjectTypes/BeetleGrass.md) when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field. NOTE: Because this property is unused in that shader, this value effectively does nothing. The intended property was likely `_ShakeWindspeed` which would have configured the speed of the shaking, but it is not configurable under normal gameplay|0.2|
|windintensity|float|Configures the range of values possible for the `_ShakeBending` shader property of a material using the MainManager.`windshader` (this property controls the amount of warping to apply to the object as a result of the wind). This shader is only used during the SetUp of a [BeetleGrass](../Entities/NPCControl/ObjectTypes/BeetleGrass.md) when MainManager.`nowindeffect` is false where the grass will get this shader as its `sprite`'s material. The shader property will have a random value that range from `X` / 2.0 to `X` where `X` is the value of this field|0.2|
|faderchange|bool|If true, all Fader will fade all their Renderer instead of culling them completely. Also, on Start, `faders` and `fss` gets their initialised on the first LateUpdate which are used in LateUpdate to reset all the sharedMaterials, shadowCastingMode and sharedMaterials's renderQueue of all disabled Fader's renders that were initially enabled|false|
|skycolor|Color|If a Fader is set to fade its GameObject's materials (forced to be the case if `faderchange` is true), this is the color tint to set the materials's colors to in RGB (the A component is ignored since Fader controls it)|Pure white|

## Fog
Maps can configure a Unity's fog effect by controlling RenderSettings.fogEndDistance (using the `fogend` configuation field) and RenderSettings.fogColor (using the `fogcolor` configuration field). The former controls the rendering distance one can see before seeing fog and the latter controls the color of the fog.

## `screeneffect`
This field has a special enum type called `ScreenEffects` and it only has 2 values possible: `None` and `SunRaysTopRight`. Only the latter value does something and it is to render a sun ray visual effect on the top right of the `GUICamera`.

## Skybox control
By default, maps won't have a skybox and instead, have pure black rendered as the background. This is because the default behavior is to have RenderSettings.skybox set to null and to have `MainCamera`'s backgroundColor set to pure black.

However, maps can define a skybox to render behind the map instead. Setting up one involves the `skyboxmat` configuration field which defines the material of the skybox to use.

The skybox material is meant to use the "Skybox/6 sided" shader which comes with Unity. This is because the map automatically manages its `_Tint` and `_Rotation` properties in the following ways:

- `_Rotation`: On FixedUpdate, if a skybox exists and `overrideskybox` is false (see the section below for details), the value of this property changes to be 180.0 + `MainCamera`'s x position. Basically, the skybox rotates depending on the horizontal position of the camera.
- `_Tint`: This property has some management done on it, but effectively, it maintains the default value being pure gray and the game can fade it to pure black when needed:
    - On Start, if there was an existing skybox, its `_Tint` gets set to pure gray when not in an [inside](Insides.md)
    - Pure gray is also the default `_Tint` value of the `skyboxmat` as it's set when setting up the skybox (if `skyboxmat` isn't null)
    - On FixedUpdate, if a skybox exists, `nocolorchange` is false (only is true in certain areas under [AreaSpecific](Init%20methods/AreaSpecific.md)) and `overrideskybox` is false (see the section below for details), the value of this property changes whenever `fadeammount` changes. This happens as a result of fading the [insides](Insides.md) when entering / exiting one, the `_Tint` value changes with `fadeammount` using a color whose r, g and b component are all `fadeammount` clamped from 0.0 to 0.5 (from pure black to pure gray). Basically, `fadeammount` changes causes the `_Tint` of the skybox to change to pure black when fading the outside and then back to pure gray again when fading out the outside

### `overrideskybox`
There is another skybox related configuration field, but it only takes effect if there is a skybox meaning that `skyboxmat` didn't had a null value when the map started.

Having this field true implies several changes in FixedUpdate:

- [insides](Insides.md) fading won't happen meaning `fadeammount` won't change and the `render` materials won't change. It also means that the `_Tint` of the skybox won't change either
- The `_Rotation` property of the skybox won't change with the `MainCamera` x position

It's essentially a configuration that prevents the skybox from having its material properties changed, but enabling it also prevents the [insides](Insides.md) fading logic to happen in general when entering or exiting an inside.

It should be noted that this setting is only used in the `SnakemouthUndergroundDoor` map and it was added as an attempt to make its skybox (`Cavern2`) look better visually since it didn't look satisfactory. However, this doesn't end up doing anything different visually because it was decided during development to simply scrap the skybox (along with `Cavern1` which had similar visual problems) and to instead mask them by adding black walls.

This setting can therefore be considered almost unused because while it is technically used and does function, the side effects related to the insides fading isn't expected. It is therefore not recommended to set this field to true.

## `globallight`
This configuration field controls RenderSetting.ambientLight. It allows the game to configure the color of the ambient lighting which is lights that gets emitted by Unity throughout the scene.

## Wind control
[BeetleGrass](../Entities/NPCControl/ObjectTypes/BeetleGrass.md) are setup with a special shader in their `sprite`'s material: MainManager.`windshader`. This shader comes with 2 properties which can be controlled by configuration fields whose value express a random range from `X` / 2.0 to `X` where `X` is the value:

- `_ShakeDisplacement` (controlled by `windspeed`): This property actually doesn't do anything because it's not used in the shader. It is very likely that the correct property should have been `_ShakeWindspeed` which would have changed the speed of the shaking, but because this one is not configurable, it's not possible to configure the shaking speed via MapControl
- `_ShakeBending` (controlled by `windintensity`): Configures the amount of warping to apply as a result of the wind shaking

## Fader control
Fader is a dedicated component that can be applied to any GameObjects on a map. It will through camera distance checking hide its GameObject by setting its Renderer's shadowCastingMode to `ShadowsOnly`. This is done because it's possible some objects could occlude the camera so much that they take a large portion of the screen. Those objects gets a Fader component so they can be hidden earlier than the camera's near clip plane culling would. This works by collecting all Renderer components in a field called `renders` and messing with all of them at once (not to be confused with MapControl's `render` which is something different).

The default of a Fader is to simply cull the GameObject when out of range, but there is another opt-in mode: fading the GameObject through a different shader (usually MainManager.`Fade3D` or MainManager.`fadePlane`) before hiding it instead of just changing the shadowCastingMode. Any maps can opt-in to this fading behavior for all Fader present by having the `faderchange` configuration field set to true. However, no matter what the value of this field is, the following [areas](../Enums%20and%20IDs/librarystuff/Areas.md) will always have this behavior:

- `FarGrasslands`
- `WildGrasslands`
- `WaspKingdom`

It's also possible for an individual Fader to force itself to operate in this mode if its `alwaysfade` field is true which is expected to be done from the map's prefab where the Fader is defined.

That being said, setting `faderchange` to true will also causes some special logic in LateUpdate to happen every 3 frames on every disabled Fader that were initially enabled when MapControl Start happened. This involves the following tracking fields:

- `faders`: All Fader found on the scene in MapControl's Start
- `fss`: All `faders`'s enablement when MapControl's Start happened

For each affected Faders, the following will happen on MapControl's Lwindspeed: 0\n
- All `renders`'s sharedMaterials's renderQueue gets collected into an array for restore later
- All `renders`'s sharedMaterials gets set to the Fader's `mats` element which essentially restores the materials to their original (Fader collects `mats` on its Start)
- All `renders`'s shadowCastingMode's gets set to the Fader's `modes` element which essentially restores them to their original (Fader collects `modes` on its Start)
- All `renders`'s sharedMaterials's renderQueue gets restored from the saved array

This effectively reverts all `renders`'s materials to their original which is desired for a disabled Fader because it forces the material to revert to their non fadable counterparts. It only applies to Faders that were initially enabled because it is assumed it's not needed to mess with them if that's the case.

### `skycolor`
On top of the fading behavior (whether or not it was caused by `faderchange` being true), it's possible to change the color of the fading done (when the fading shader used is MainManager.`Fade3D`). It can be done through changing the `skycolor` configuration field. Since the Fader controls the alpha component of the color, only the r, g and b components of this field will be used. If the fading mode of the Fader isn't used, the `skycolor` configuration field won't do anything.

There is an override on this field on [AreaSpecific](Init%20methods/AreaSpecific.md): if the [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `WaspKingdom` and `skycolor` converted to Vector3 has a magnitude above 0.9, it is overriden to pure gray.
