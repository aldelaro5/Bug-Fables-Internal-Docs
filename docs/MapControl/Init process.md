# MapControl init process
The init process involves 2 phases:

- Start
- The first LateUpdate (known as `latestart`)

## Start
Here is what Start does:

- Reset ScrewPlatform.`camischanging` to false so the camera won't be fixed somewhere else as the map initialises
- MainManager.FixSamira called. For more information, consult the [samira section](../General%20systems/Music%20playback.md#checksamira) section of the music playback system
- Reset the DeadLanderOmega state (`state` to 0, `detected` to false, `activeid` to -1 and `hand` to null)
- If the skybox is present and we are not in an inside, the skybox's shader `_Tint` is set to pure gray

Camera gets initialised with the following fields (the default value is picked if the field's magnitude is 0.2 or below):

|Value|Field|Default|
|----:|-----|-------|
|instance.`camoffset`|`camoffset`|`defaultcamoffset`|
|instance.`camangleoffset`|`camangle`|`defaultcamangle`|

- `mainmesh` gets a default value of the first child object if it was null
- `mainrender` gets a default value of the `mainmesh`'s Renderer if it was null
- `musicpreload` set to a new list
- If `areaid` isn't the current one, MainManager.UpdateArea is called with the new area. For more information, check the [areaid section](Map%20identification.md#areaid) of the map identification documentation
- `actualcenter` set to `centralpoint`
- If there are any `canfollowID` elements, `tempfollowers` is initialised to a new list
- instance.`battlestage` set to `battlemap` (this becomes the default stage to use on StartBattle if the stageid is -1)
- The TextAsset of the map's dialogues is located and loaded. For more details on how this is done, check the [`dialogues` section](SetText%20configuration.md#dialogues) of the SetText configuration documentation
- If the dialogues file loaded sucessfully, `dialogues` is set to its data whose endlines are normalised to LF and split by LF
- If `useglobalcommand` is true, the [GlobalCommand](SetText%20configuration.md#global-commands-system) system is enabled. Check the documentation to learn more
- [CreateEntities](Init%20methods/CreateEntities.md) called which initialises `entities`
- The skybox gets set:
    - If `skyboxmat` exists, skybox gets set to it and if we aren't in an inside, its shader `_Tint` gets set to pure gray
    - Otherwise, skybox gets set to null and the `MainCamera`'s backgroundColor gets set to pure black
- GetSkyColor called which just sets `skycolor`'s alpha to 1.0 (fully opaque)
- If not `inevent` and `keepmusic` is false, [Music](Init%20methods/Music.md) is called which sets the music and initialises some music fields
- CheckDisc is invoked in 1.0 seconds which will set `hiddenitem` to 100 (only the fact it's not null matters) which will allow the `Detector` [medal](../Enums%20and%20IDs/Medal.md) to beep under specific conditions. Check the [discoveryids](Miscellaneous%20features.md#discoveryids) documentation to learn more as this array is heavily involved in the checks for this
- All `flowerbed` of all `playerdata` entities gets destroyed if any exists
- `render` gets set to all MeshRenderer under in the children of `mainmesh`
- `ogmat` is set to all materials array of all `render`
- `mapflags` is initialised to 10 elements
- GetDigWalls called which sets `digwall` with the following:
    - All GameObjects with `DigWall` tag are found and all their first Collider are added to `digwall`
    - All GameObjects with `Respawn` tag have their layer set to 0 (Default) and if they have a BoxCollider, it is set to be trigger

The fog settings are configured with fields:

|Value|Field|
|----:|-----|
|fogEndDistance|`fogend`|
|fogColor|`fogcolor`|
|ambientLight|`globallight`|

- SetScreenEffect called which will only do something if `screeneffect` is `SunRaysTopRight`. If it is, a new `Prefabs/Particles/SunRay` is instantiated childed to the `GUICamera` with a local position of (9.0, 7.0, 10.0) without rotations
- [RefreshInsides](Insides.md#refreshinsides) is called without inside and without caller which forces the initial state of the map to be outside
- [CheckQuests](../TextAsset%20Data/BoardQuests%20data.md#checkquests) called
- RefreshEntities called with forceanim and refreshmap
- CheckAchievement called which updates the [records](../Enums%20and%20IDs/librarystuff/Records%20entry.md)'s states
- [AreaSpecific](Init%20methods/AreaSpecific.md) called
- If `insidetypes` length isn't the `insides` one, `insidetypes` gets truncated to have a matching length
- The shader global `GlobalIceRadius` is set to 0.0
- If `mapid` is `BugariaResidential`, CombineMesh is invoked in 0.1 seconds. Check the [merged mesh](Merged%20mesh.md) documentation to learn more

## First LateUpdate (`latestart`)

- `tempfollowers` gets updated from instance.`extrafollowers` using `canfollowID` and `chompy` gets set if needed. Check the [follower system](Follower%20system.md) documentation to learn more
- SetParticles called. See the [particles level](Particles%20level.md) documentation to learn more
- PreloadSprites is invoked in 0.1 seconds which will set `entitysprite` to each of `entities`'s `sprite`'s texture
- If `faderchange` is true:
    - `faders` is set to all Fader in this map
    - `fss` is set to all the enablement of all `faders`
- SetPlayerColliders is invoked in 0.2 seconds. See the [entityonly](EntityOnly.md) documentation to learn more
- `latestart` is set to true which blocks this logic from happening again
