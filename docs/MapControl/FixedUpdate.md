# FixedUpdate

- If `tieYtoplayer` is true while instance.`camtarget` isn't null (the camera is following a transform):
    - `actualcenter` is set to `centralpoint`, but the y is instance.`camtarget` y position
    - If `tetherYLerp`.x is higher than 0.0, `tetherdistance` is set to a lerp from `tetherYLerp`.x to `tetherYLerp`.y with a factor of instance.`camtarget` y position / `tetherYLerp`.z TODO: ???
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
    - All `render` who isn't null, doesn't have a `NoMapColor` tag and whose index exists in `ogmat` has their sharedMaterials set to their matching `ogmat` TODO: ???
    - `refreshmats` is set to false

