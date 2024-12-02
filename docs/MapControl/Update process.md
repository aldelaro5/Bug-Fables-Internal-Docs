# MapControl update process
The Update process consists of 2 parts:

- LateUpdate (excluding the first call so only when `latestart` is strue)
- FixedUpdate

## LateUpdate
Nothing happens if the player doesn't exist.

- If not in a `minipause`, `pause`, `inevent` or `message`:
    - If `autoevent` exists, they are processed. Check the [`autoevent`](Miscellaneous%20features.md#autoevent) documentation to learn more
    - If `hiddenitem` isn't null, the detector emote is set with player.entity.`emoticonid` set to 4 with an `emoticoncooldown` of 100.0. `hiddenitem` is set to null after. This sets up the `Detector` [medal](../Enums%20and%20IDs/Medal.md) emoticon. The sound will be played a bit later
- CheckStencilSwitch called which finds the first StencilSwitch that's not `iskill` and is `hit`:
    - If one is found:
        - `GlobalIceRadius` shader global property gets set to the StencilSwitch's `internaltransform[0]` scale magnitude / 2.0
        - `CentralIcePoint` shader global property gets set to the StencilSwitch's position
        - `stencilid` gets set to the map entity id of the StencilSwitch
    - Otherwise (none is found):
        - `GlobalIceRadius` shader global property gets set to a lerp from it to 1/20 of the game's frametime ???
        - `stencilid` gets set to -1
- If the detector sound isn't playing, but if the detector emote is playing still (player.entity.`emoticonid` is 4 and `emoticoncooldown` hasn't expired yet), `Select1` sound is played on `sounds[11]` with 1.1 pitch and 0.25 volume
- All `digwall`'s enablement gets updated. See the [digwall](DigWall.md) documentation to learn more

What happens next depends on [`alivetime`](Miscellaneous%20features.md#alivetime) since it can prevent some logic to be done.

### `alivetime` is higher than 0
It is decreased by 1.0 and no further logic happens from here. Essentially, it blocks all the logic that would have happened below for a certain amount of frames including the first one.

### `alivetime` is 0

- Every 2 frames, NPCControl updates happen for any that have their `requires`, `limit` and `regionalflag` condition fufilled to exist:
    - [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md): The enablement is updated to be only enabled only if the player is less than 15.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this, the `emoticon` of the entity is disabled
    - [NPC](../Entities/NPCControl/NPC.md) (with an `originalid` defined and only if the map's `keepobjectsactive` is false): The enablement is updated to be only enabled only if the player is less than the NPC's `radius` * 2.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this and `interacttype` isn't `Talk`, the `emoticon` of the entity is disabled

## FixedUpdate

- If `tieYtoplayer` is true while instance.`camtarget` isn't null (the camera is following a transform):
    - `actualcenter` is set to `centralpoint`, but the y is instance.`camtarget` y position
    - If `tetherYLerp`.x is higher than 0.0, `tetherdistance` is set to a lerp from `tetherYLerp`.x to `tetherYLerp`.y with a factor of instance.`camtarget` y position / `tetherYLerp`.z. NOTE: This feature is complicated, see the [tetherYLerp details](Camera%20system.md#more-details-on-tetherylerp) section in the camera system page for more details
- If `overrideskybox` is false and RenderSettings.skybox isn't null:
    - If `nocolorchange` is false, the [UpdateInsideColor](Insides.md#outside-fading) logic happens here with a targetalpha of 1.0 if not in an inside or 0.0 if in an inside. Check the documentation to learn more
    - The `_Rotation` material property of the skybox is set to 180.0 + MainCamera x position
- If `rotatecam` is true, there are `roundways` and we are not in an inside, `roundways[0]` position is set to ((0.0 - MainCamera's forward x vector) * 3.0, 5.0, (0.0 - MainCamera's forward z vector) * `lightoffset`). NOTE: This feature is very specific to a the `AntTunnels` [map](../Enums%20and%20IDs/Maps.md), check the [`roundways[0]`](Camera%20system.md#roundways0) documentation to learn more. Also, this math is incorrect

