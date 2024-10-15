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
|tetherYLerp|Vector3|Only applies if `tieYtoplayer` is true and `camtarget` isn't null. This field contains the 3 parameters to a lerp operation to do on FixedUpdate to update `tetherdistance` if the x component is above 0. The lerp will be done from the x component to the y component with a factor of `camtarget` y position / the z component TODO: what does this do intuitively?|Vector3.zero|
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

As for `tetherdistance`, while a constant value can be set in the prefab, it can get overriden by FixedUpdate and leave its management to `tetherYlerp` if its x component is above 0.0. How it works is on each FixedUpdate where `tieYtoplayer` is true and instance.`camtarget` isn't null, `tetherdistance` is set to a lerp from `tetherYLerp`.x to `tetherYLerp`.y with a factor of instance.`camtarget` y position / `tetherYLerp`.z. TODO: explain intuitively what this does

### `roundways[0]`
TODO: need to intuitively understands how this works before explaining it
