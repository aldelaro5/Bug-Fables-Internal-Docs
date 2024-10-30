# MapControl update process

## LateUpdate
Nothing happens if the player doesn't exist.

### Non `latestart`

- If not in a `minipause`, `pause`, `inevent` or `message`:
    - If `autoevent` exists, they are processed in order. Each x component correspond to a flag that must be false for the event id y to be started immediately without caller. If event y was started, its x flag slot is set to true (this means multiple events could start at the same time due to this)
    - If `hiddenitem` isn't null, the detector emote is set with player.entity.`emoticonid` set to 4 with an `emoticoncooldown` of 100.0. `hiddenitem` is set to null after
- CheckStencilSwitch called which finds the first StencilSwitch that's not `iskill` and is `hit`:
    - If one is found:
        - `GlobalIceRadius` shader global property gets set to the StencilSwitch's `internaltransform[0]` scale magnitude / 2.0
        - `CentralIcePoint` shader global property gets set to the StencilSwitch's position
        - `stencilid` gets set to the map entity id of the StencilSwitch
    - Otherwise (none is found):
        - `GlobalIceRadius` shader global property gets set to a lerp from it to 1/20 of the game's frametime ???
        - `stencilid` gets set to -1

### Common logic

- If the detector sound isn't playing, but if the detector emote is playing still (player.entity.`emoticonid` is 4 and `emoticoncooldown` hasn't expired yet), `Select1` sound is played on `sounds[11]` with 1.1 pitch and 0.25 volume
- All `digwall`'s enablement gets updated with the player.`digging` value: if the player is `digging`, they are disabled and if the player isn't `digging`, they are enabled

### `alivetime` is higher than 0
It is decreased by 1.0 and no further logic happens from here. Essentially, it blocks all the logic that would have happened below for a certain amount of frames including the first one.

### `alivetime` is 0

- Every 3 frames, if `faderchange` is true and there `faders`, each enabled one whose `fss` is true (it started enabled) has the following happen to them:
    - The [Fader logic](Graphics%20configuration.md#fader-control) happens
- Every 2 frames, NPCControl updates happen for any that have their `requires`, `limit` and `regionalflag` condition fufilled to exist:
    - DoorOtherMap: The enablement is updated to be only enabled only if the player is less than 15.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this, the `emoticon` of the entity is disabled
    - NPC (with an `originalid` defined and only if the map's `keepobjectsactive` is false): The enablement is updated to be only enabled only if the player is less than the NPC's `radius` * 2.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this and `interacttype` isn't `Talk`, the `emoticon` of the entity is disabled

## FixedUpdate

- If `tieYtoplayer` is true while instance.`camtarget` isn't null (the camera is following a transform):
    - `actualcenter` is set to `centralpoint`, but the y is instance.`camtarget` y position
    - If `tetherYLerp`.x is higher than 0.0, `tetherdistance` is set to a lerp from `tetherYLerp`.x to `tetherYLerp`.y with a factor of instance.`camtarget` y position / `tetherYLerp`.z. NOTE: This is complicated, see the [tetherYLerp details](Camera%20system.md#more-details-on-tetherylerp) section in the camera system page for more details
- If `overrideskybox` is false and RenderSettings.skybox isn't null:
    - If `nocolorchange` is false, the skybox and `render`'s material's color are updated when entering or exiting an inside (see the section below)
    - The `_Rotation` material property of the skybox is set to 180.0 + MainCamera x position
- If `rotatecam` is true, there are `roundways` and we are not in an inside, `roundways[0]` position is set to ((0.0 - MainCamera's forward x vector) * 3.0, 5.0, (0.0 - MainCamera's forward z vector) * `lightoffset`)

## Updating skybox and `render` material colors with inside state

- If `fadeammount` is more than 0.025 away from 1.0 (0.0 instead if we are in an inside):
    - `refreshmats` is set to true
    - `fadeammount` is set to a lerp from the existing one to 0.0 (1.0 instead if in an inside) with a factor of TieFramerate of `fadingspeed`
    - The `_Tint` material property of the skybox is set a color where r, g and b are `fadeammount` clamped from 0.0 to 0.5
    - If `render` exists, each `render` element that isn't null has their enablement changed to true if they have the `AlwaysShow` tag. If they don't, it's set to true of `fadeammount` is above 0.15, false otheriwse. Additionally, if it doesn't have the tag, every materials has their color set to a color where r, g and b are `fadeammount` if they fufill all of the following conditions:
        - Their shader isn't the `emptymat`, `outlinemain` or `fakelight` shader
        - The `NoMapColor` isn't present on the matching `render`
        - The material has a `_Color` property
- Otherwise, if `render` exists, we aren't in an inside and `refreshmats` is true:
    - All `render` who isn't null, doesn't have a `NoMapColor` tag and whose index exists in `ogmat` has their sharedMaterials set to their matching `ogmat`
    - `refreshmats` is set to false

