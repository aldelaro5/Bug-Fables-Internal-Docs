# MapControl init process
TODO

## Start

- Reset ScrewPlatform.`camischanging` to false so the camera won't be fixed somewhere else as the map initialises
- MainManager.FixSamira called which removes the following Musics from instance.`samiramusics`:
    - `Title`
    - `Wind`
    - `Water`
    - `MachineHum`
    - `Breathing`
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
- If `areaid` isn't the current one, MainManager.UpdateArea is called with the new area:
    - If exiting area 15 (Giant's Lair), and flag 596 is true (spoke to the Roach Elder in postgame), flag 597 is set to true (changed area after speaking to the Roach Elder in postgame)
    - All regionals are reset
    - instance.`areaid` is updated to the new area
    - The `librarystuff` flag of the area is set to true
    - UpdateShops gets called which will randomly shuffle all `availablebadgepool` arrays (this changes the items on the shelf of the shops)
- `actualcenter` set to `centralpoint`
- If there are any `canfollowID` elements, `tempfollowers` is initialised to a new list
- instance.`battlestage` set to `battlemap` (this becomes the default stage to use on StartBattle if the stageid is -1)
- The TextAsset of the map's dialogues is located and loaded with the following logic (all from the ressources directory):
    - If the map is `TestRoom`, the file is `Data/TestRoom`
    - If it's not, the file path starts with `Data/DialoguesX/Maps/` where `X` is the `languageid`. As for the actual filename within this directory:
        - If `readdatafromothermap` is set to `TestRoom` (which means it was left at its default value), it's the string representation of `mapid`, otherwise, it's the `readdatafromothermap` one
- If the dialogues file loaded sucessfully, `dialogues` is set to its data whose endlines are normalised to LF and split by LF
- If `useglobalcommand` is true, `commandlines` is set to a file within `Data/Commands/` from the ressources directory. As for which file, if `readdatafromothermap` is set to `TestRoom` (which means it was left at its default value), it's the string representation of `mapid`, otherwise, it's the `readdatafromothermap` one
- CreateEntities called which initialises `entities`
- The skybox gets set:
    - If `skyboxmat` exists, skybox gets set to it and if we aren't in an inside, its shader `_Tint` gets set to pure gray
    - Otherwise, skybox gets set to null and the `MainCamera`'s backgroundColor gets set to pure black
- GetSkyColor called which just sets `skycolor`'s alpha to 1.0 (fully opaque)
- If not `inevent` and `keepmusic` is false, Music is called which sets the music and initialises some music fields with the following:
    - If there's no `music`, ChangeMusic is called with null (silence) at 0.1 fadespeed
    - Otherwise:
        - If there's `musicflags`, `musicid` is set to -1 followed by the processing of each music flag in reverse order:
            - The first one with an x of -1 or an x of a flags slot that is true will be selected. Being selected means `musicid` will be set to the y component and the processing is over
        - If `musicid` isn't negative, ChangeMusic is called with `music[musicid]` with 0.1 fadespeed followed by a CheckSamira call on `music[musicid]`
        - Otherwise, ChangeMusic is called with null (silence)
- CheckDisc is invoked in 1.0 seconds which will set `hiddenitem` to 100 (TODO: ???) if all of the following conditions are fufilled:
    - There's at least 1 `discoveryids`
    - The `Detector` medal is equipped
    - If `mapid` is `TermiteIndustrial`, the player's z position must be lower than 20.0 (meaning they need to be far back enough to be in front of the Colosseum's entrance)
    - At least one `discoveryids` is a discovery id that hasn't been obtained yet
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
- RefreshInsides is called without inside and without caller TODO: ???
- CheckQuests called TODO: ???
- RefreshEntities called with forceanim and refreshmap
- CheckAchievement called which updates the records's states
- AreaSpecific called TODO: ???
- If `insidetypes` length isn't the `insides` one, `insidetypes` gets truncated to have a matching length (but the values are left at default) TODO: unused ???
- The shader global `GlobalIceRadius` is set to 0.0
- If `mapid` is `BugariaResidential`, CombineMesh is invoked in 0.1 seconds which will do the following (this combines all `Merge` GameObject's meshes including their MeshCollider into one GameObject childed to the first one):
    - Gather all GameObjects with tag `Merge`, if none exists, the method does nothing
    - Build an array of CombineInstance where each mesh is each `Merge` GameObject's MeshFilter and each transform is the GameObject's world transform. During the course of building the array, all `Merge` GameObject are destroyed except the first one, but it will have its MeshRenderer destroyed (its materials will be stored)
    - Create a new GameObject named `merged mesh` with a MeshFilter containing a new Mesh then having CombineMeshes called on it with that new mesh using the CombineInstance array built earlier. The GameObject is childed to the first `Merge` GameObject with angles (-90.0, 180.0, 0.0) and uses layer 8 (`Ground`). It has a MeshRenderer using the materials stored from the first `Merge` GameObject. Fionally, it has a MeshCollider with a sharedMesh set to the MeshFilter's mesh
    - The MeshRenderer just added is added to `render`

## First LateUpdate (`latestart`)

- `tempfollowers` gets updated from instance.`extrafollowers`:
    - If the object's name is `0` (only the case with the `TestRoom` map), MainManager.AddFollower is called with instance.`extrafollowers` without caller (so every followers is allowed)
    - Otherwise, only the ones with an animid present in `canfollowID` will be added. When each is added, their `tempofollowerid` gets updated to the found `cantfollowID` index and their `onground` is set to false. If the animid is `Maki`, their `ccol` height is set to 3.0 and center to 1.5 in y. NOTE: `tempfollowerid` isn't set correctly because it's supposed to be `tempfollowers` index, NOT `canfollowID` index
- If flag 402 is true (Chompy is with Team Snakemouth) and the player isn't in a `submarine`:
    - MainManager.AddFollower is called with animid 169 (`ChompyChan`) without caller
    - `chompy` is set to the follower that was just added, their `onground` is set to false and their `tempfollowerid` is set to the matching `tempfollowers` index
- SetParticles called which, if MainManager.`particlelevel` is 0, will look for any GameObject with the `Particable` tag and disable all of them
- PreloadSprites is invoked in 0.1 seconds which will set `entitysprite` to each of `entities`'s `sprite`'s texture
- If `faderchange` is true:
    - `faders` is set to all Fader in this map
    - `fss` is set to all the enablement of all `faders`
- SetPlayerColliders is invoked in 0.2 seconds which will find all the first collider of all GameObject with the tag `EntityOnly` and assign them to `entityonly`. These colliders gets their staticFriction and dynamicFriction set to 0.0 and all `playerdata` and `tempfollowers` entities ignores these colliders
- `latestart` is set to true which blocks this logic from happening again

