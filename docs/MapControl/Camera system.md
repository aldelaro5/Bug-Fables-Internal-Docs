# Map camera system
The [camera system](../General%20systems/Camera%20system.md) is greatly influenced by MapControl and it can be configured with the following [configuration fields](Fields/Configuration%20fields.md):

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
|tetherYLerp|Vector3|Only applies if `tieYtoplayer` is true and `camtarget` isn't null. This field contains the 3 parameters to a lerp operation to do on FixedUpdate to update `tetherdistance` if the x component is above 0. The lerp will be done from the x component to the y component with a factor of `camtarget` y position / the z component. NOTE: This feature is complicated, see the section below for details|Vector3.zero|
|roundways|Transform[]|Only the first element matters. If the first element exists, `rotatecam` is true and we aren't in an [inside](./Insides.md), the corresponding Transform will be positioned relative to the camera's forward vector on FixedUpdate in x and z (the y component is always 5.0). TODO: explain intuitively how this works The position is multiplied by `lightoffset` before being set|Empty array|
|lightoffset|float|If `roundways[0]` exists, `rotatecam` is true and we aren't in an inside, this field will be a multiplier applied to the position to set `roundways[0]` in FixedUpdate|5.0|

Most of the camera system is actually managed by MainManager via some fields that act as a state machine and via the RefreshCamera method which is invoked on FixedUpdate (after LoadEverything is done).

However, the current map is what has the biggest influence on the camera initially so the parts that are specific to MapControl will be documented here.

## Main [Camera system](../General%20systems/Camera%20system.md) influence
There are several parts MapControl influences of the main camera system.

### Initial position and angles
The initial values of instance.`camoffset` and instance.`camangleoffset` are dictated by the MapControl's `camoffset` and `camangle` fields. This gives a starting default position, but they will default to their respective values if the prefab didn't had values set for them.

### Camera limits
MapControl also allows to restrict the camera's positioning during RefreshCamera. The only requirement to make use of this feature is the current MapControl isn't null (which is only the case when loading a map).

There are 2 types of limits that can be enforced. They stack and are applied in the order mentioned (this doesn't include the local position displacement which is done after both of these):

- Hard positional limit: This limits the camera to have a position that always falls between `camlimitneg` and `camlimitpos`. It's a very hard limit: the camera will never be allowed to go beyond this range
- Radius limit: This limits only applies if `tetherdistance` is above 0.0 (it has a value set) and we aren't in an [inside](./Insides.md). This limits the camera position to fall inside a circle where the center is `actualcenter` and the radius is `tetherdistance`

#### More details on the radius limit
About `actualcenter`, while the intial value of `actualcenter` is `centralpoint` which is a configuration field, its y component can get overriden by FixedUpdate to instance.`camtarget` y position if `tieYtoplayer` is true and `camtarget` isn't null.

Basically, by setting `tieYtoplayer` to true, it also configures the radius limit to ignore the y component if the camera is set to move towards instance.`camtarget` since they'll be the same in this limit check so only the x and z component matters.

As for `tetherdistance`, while a constant value can be set in the prefab, it can get overriden by FixedUpdate and leave its management to `tetherYlerp` if its x component is above 0.0. How it works is on each FixedUpdate where `tieYtoplayer` is true and instance.`camtarget` isn't null, `tetherdistance` is set to a lerp from `tetherYLerp`.x to `tetherYLerp`.y with a factor of instance.`camtarget` y position / `tetherYLerp`.z.

`tetherYLerp` however is a relatively complicated features that warrants more explanation of what it tries to do.

#### More details on `tetherYLerp`
This configuration field essentially is an attempt to address a simple problem that technically was already solved. In a `rotatecam` map with `tieYtoplayer` enabled, the camera will want to LookAt the `actualcenter` of the map, but it will do so once its position and offsetting were updated. This means all of this happens AFTER the `camlimitneg`, `camlimitpos` and `tetherdistance` constraints were processed.

The problem comes with a map that fits those conditions, but is also layed out vertically. This is the case particularily with the `WizardTowerStairs` [map](Init%20methods/AreaSpecific.md) which fits all of the above to run into this problem.

The problem involves the `tetherdistance` contraints as the `camtarget` (usually the `player`)'s y position. As it increases, it means the `camtarget` is going further and further away from the `map`'s `actualcenter` so it means that the distance eventually gets larger than `tetherdistance` which will block the camera from going upward as the player goes up. The result is the camera will be forced the look downward because it still has to follow its `camtarget`, but is forced to look down towards the map's `actualcenter` instead of having its y follow its target.

However, this problem would have already been solved: the game does actually updates `actualcenter`'s y to match the `camtarget`'s y in FixedUpdate when `tieYtoplayer` is enabled which it is in this case. The reason this fix doesn't work is because for some reason, the `camlimitneg`'s y and `camlimitpos`'s y of this map are set to 0.0. This forces the camera's y to be 0.0 and unfortunately, this limit applies BEFORE the `tetherdistance` one. Please note that this is separate from `camoffset` which is free to be applied after all limits so `camoffset` ignores those limits.

Since the camera is forbidden to go up, but the `actualcenter`'s y follow the player, this creates the exact same y difference that will incorrectly impact the `tetherdistance` check. Effectively, due to the map's misconfiguration, it has unfixed the problem's fix.

Instead of correctly configuring the map, what was done was apply a workaround which is what `tetherYLerp` is. What this feature does is to allow `tetherdistance` to progressively change from a value to another as the `camtarget`'s y changes. All of this is done with a lerp between the 2 desired values where the factor is the `camtarget`'s y / a third configurable number.

This essentially allows to compensate this issue by increasing `tetherdistance` as the `camtarget`'s y increases. It is an improper way to perform this workaround because it now allows the camera to go beyond the desired bounds in x and z, but in practice, this doesn't happen because the `tetherYLerp` configured for this map happens to tuned well enough to not let the camera go too far as well as keeping the default `camoffset`.

Other than `WizardTowerStairs`, the only other map that technically uses this feature is `MysteryIsland`, but this one has it missconfigured because the y and z components are left at 0.0. This leads to unexpected behaviors such as the lerp factor being a division by 0.0, but in practice, this feature doesn't do anything because the `tetherYLerp.x` is too high to be impactful and even if it was disabled entirely, it wouldn't have restricted the camera in a meaningful way.

In the end, it is not recommended to use this feature and to instead configure the map properly. The `actualcenter`'s y value already correctly follows the `camtarget`'s y so it is not necessary to apply such a questionable workaround.

### `roundways[0]`
TODO: need to intuitively understands how this works before explaining it
