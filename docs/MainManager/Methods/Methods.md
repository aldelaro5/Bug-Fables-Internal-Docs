# Methods

## Math and Unity utilities

```cs
public static bool IsWithin(int value, int min, int max)
```
Returns true if `value` is both greater than or equal `min` and less than or equal `max`, false otherwise.

```cs
public static void PushAway(Transform obj, Vector3 otherobj)
public static void PushAway(Transform obj, Vector3 otherobj, float value)
```
Adds a vector to the `obj`'s postion corresponding to `obj`'s position - otherobj all normalised and multiplied by `value` (this essentially moves `obj` to be away from `otherobj` with a distance of `value`). On the overload without a `value` parameter, the value defaults to 0.05.

```cs
public static void LaunchObject(Transform obj, Vector3 push)
public static void LaunchObject(Rigidbody r, Vector3 push, bool gravity)
```
Set some fields of `r` to launch `r` or `obj`'s RigidBody:

- velocity: `push`
- isKinematic: false
- useGravity: `gravity`

On the overload without a `gravity` parameter, the default value is true.

```cs
public static void RandomSort(ref List<int> array)
public static void RandomSort(ref int[] array)
```
Changes `array` such that it is randomly shuffled using the Fisher–Yates algorithm. On the `List<int>` overload, `array` is set to a newly created list contained the shuffled elements.

```cs
public static int HowManyTrue(bool[] array)
```
Returns the amount of elements in `array` that have a value of true.

```cs
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limitspos, Vector3 limitsneg)
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limits)
```
Returns a vector where each component of `input` got clamped from the matching component of `limitneg` to the matching component of `limitspos`.

On the overload taking a `limits` parameter, `limitspos` becomes `limits` and `limitsneg` becomes -`limits`.

```cs
public static Vector3 ClampMagnitude(Vector3 v, float max, float min)
```
Returns a vector that matches `v` in direction, but its magnitude got clamped from `min` to `max`. More precisely, the vector that gets returned depends on the following conditions:

- If `v`.sqrMagnitude is higher than `max` * `max`, `v`.normalized * `max` is returned
- If `v`.sqrMagnitude is lower than `min` * `min`, `v`.normalized * `min` is returned
- Otherwise, `v` is returned unchanged

```cs
public static IEnumerator LerpObject(Transform obj, Vector3 position, float speed, bool destroyonend)
```
Moves `obj` via a Lerp from its current position to `position` with a factor that grows from 0.0 to 1.0 at a rate of TieFramerate(`speed`) each frame. Once the Lerp completed, `obj` gets destroyed if `destroyend` is true.

```cs
public static IEnumerator Spin(Transform obj, Vector3 target, float frametime, bool smooth)
```
Changes `obj` angles towards `target` over the course of `frametime` + 1.0 amount of frames via a Lerp. If `smooth` is true, a SmoothLerp is used instead.

```cs
public static IEnumerator TempColor(Color color, float frametime, SpriteRenderer render)
```
Smoothly changes the material.color of `render` towards `color` over the course of `frametime` + 1.0 amount of frames, yields an additional frame and then set the `render`'s color back to where it was before this coroutine.

This coroutine is meant to be stored in `templetter` because it is set to null at the end of this coroutine so this can be used to track its progress.

```cs
public static IEnumerator GradualColor(Renderer obj, Color target, float frametime)
public static IEnumerator GradualColor(SpriteRenderer obj, Color target, float frametime, bool sprite)
```
Changes `obj`.material.color (or `obj`.color for a SpriteRenderer) towards `target` via a Lerp over the course of `frametime` + 1.0 amount of frames. The `sprite` parameter is unused.

```cs
public static float ClampedAngle(float input, float max)
```
This is UNUSED and is considered broken. If called, it performs the following:

- If `input` is negative, `max` is added to it
- As long as `input` is higher than `max`, `input` is set to `max` - `input`
- `input` is returned

```cs
public static bool Interval(float value, float min, float max, bool inclusive)
```
This is UNUSED, but remains functional.

Returns true if `value` is between `min` and `max`, false otherwise. If `inclusive` is true, a `value` of exactly `min` or `max` will count as being inside the interval while if `inclusive` is false, it will not count as being inside.

```cs
public static Vector2 GetPressedDirection(bool hold)
```
This is UNUSED, but remains functional. Returns a normalized vector that represents the accumulation of all 4 cardinal directions that are being pressed with up/down and left/right separation. This results in the following logic to determine the components of the vector tested in order (the vector gets normalised after all of this):

- x: -1.0 for left, 1.0 for right, 0.0 for neither
- y: 1.0 for up, -1.0 for down, 0.0 for neither

```cs
public static float DimishingReturns(float input, float decay)
```
This is UNUSED, but remains functionsl. Returns (`input` - 1.0) * `decay`.

```cs
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius)
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius, bool ignoreY)
```
Returns a vector that is `origin` + a vector whose y component is `pos`.y and the x/z components are the ones obtained from a ClampMagnitude vector of `pos` - `origin` with a maxLength of `radius`. If `ignoreY` is true, the resulting vector's y component is always `pos`.y instead of being `origin`.y + `pos`.y.

The first overload doesn't implement the `ignoreY` logic so it's as if its value was false.

The intuitive way to see this method is that the return is `pos`, but limited by magnitude to be within a circle with an origin of `origin` and a radius of `radius`. Having `ignoreY` as true means the y componet of `pos` won't apply to this radius limit.

```cs
public static Vector3 LerpVectorAngle(Vector3 input, Vector3 target, float ammount)
```
Returns a vector where each of its component is the result of a LerpAngle from `input` to `target` with a factor of `ammount` using matching components for all vectors.

```cs
public static Vector3 LerpVectorAngleSmooth(Vector3 input, Vector3 target, Vector3 current, float ammount)
```
This is UNUSED, but remains functional. Returns a vector where each of its component is the currentVelocity output of a SmoothDampAngle from `input` to `target` with a smoothTime of `ammount` using matching components for all vectors.

```cs
public static float QuadraticY(float x1, float x2, float ymax, float currentx)
public static float QuadraticY(Vector2 x1, Vector2 x2, float ymax, float currentx)
```
Returns the y component of the vector given by BeizierCurve(`x1`, `x2`, `ymax`, GetPercentage(`x1`.x, `x2`.x, `currentx`)). Intuitively, it's the same as a BeizierCurve, but the factor is expressed in terms of the x of the desired point to compute to get its y.

The first overload is UNUSED, but remains functional where `x1` and `x2` becomes vector where their x component is the matching float sent and their y/z components are 0.0.

```cs
public static Vector2 BeizierCurve(Vector2 start, Vector2 end, float ymax, float t)
```
Compute and return the point on a quadratic bezier curve using `t` as the factor going from `start` to `end` with a control point located at the midpoint of `start` and `end` in x, but with a y component of `ymax`. Intuitively, it's a simplified representation of a bezier curve that only allows to tune the y component of the control point to specify how steep the curve should go from `start` to `end`.

NOTE: `t` isn't clamped from 0.0 to 1.0, but only values in these ranges are considered valid.

```cs
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, Vector3 mid, float t)
```
Compute and return the point on a quadratic bezier curve using `t` as the factor going from `start` to `end` with a control point of `mid`. `t` is always clamped from 0.0 to 1.0 before the computation.

```cs
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, float ymax, float t)
```
Compute and return the point on a quadratic bezier curve using `t` as the factor (always clamped to 0.0 to 1.0 before the computation) going from `start` to `end` with a control point that has the following components:

- x: midpoint of `start` and `end`'s x components
- x: Dpends on `ymax`:
    - Higher than 0.0: `ymax` + midpoint of `start` and `end`'s y components
    - 0.0 or lower: `ymax`
- z: midpoint of `start` and `end`'s z components

Intuitively, this overload offers a simplified way to compute a bezier curve where only the y offset from the midpoint between `start` and `end` is configurable which only configures how steep the curve goes in y.

```cs
public static float BeizierFloat(float height, float t)
```
Compute and return the y value on a quadratic bezier curve using `t` as the factor going from Vector3.zero to Vector3.zero with a control point of (0.0, `height`, 0.0). `t` is always clamped from 0.0 to 1.0 before the computation. Intuitively, this allows to compute movement purely in the y axis, but without needing to change the x or z components so it's ideal for y axis only movement.

```cs
public static float QuadraticYSemiClamped(float currentx, float startx, float endx, float modifier)
```
This is UNUSED. Returns the y value of a quadratic curve expressed like (x - `endx`)(x - `start`) * a where x is `currentx` and a is 0.0 - (`modifier` + GetDistance(`startx`, `endx`) / 10.0) / GetDistance(`startx`, `endx`). Intuitively, `startx` and `endx` represents the 2 zeroes of the quadratic curve and `currentx` represents the x of the point we want to get its y. `modifier` influences the scaler of the curve, but the math doesn't make sense.

```cs
public static float QuadraticYUnclamped(float currentx, float startx, float endx, float modifier)
```
Returns the y value of a quadratic curve expressed like (x - `endx`)(x - `start`) * a where x is `currentx` and a is 0.0 - `modifier`. Intuitively, `startx` and `endx` represents the 2 zeroes of the quadratic curve, `currentx` represents the x of the point we want to get its y and `modifier` is a scaler to the curve. To have an unscaled quadratic curve, `modifier` needs to be -1.0.

```cs
public static Vector3 RandomVector(Vector3 input)
public static Vector3 RandomVector(float randomx, float randomy, float randomz)
public static Vector3 RandomVector(float input)
public static Vector3 RandomVector(Vector2 input)
public static Vector3 RandomVector(float randomx, float randomy)
```
Returns a vector where each components is determined randomly between -`input` and `input` with matching components for all vectors involved.

For the second overload, it's as if the `input` vector was (`randomx`, `randomy`, `randomz`).

For the third overload, it's as if each components of `input` was the float value sent.

The fourth overload is UNUSED, but remains functional and it simply doesn't generate a vector with a z component, but otherwise works the same way.

The fifth overload behaves similarily to the second overload, but it doesn't generate a vector with a z component.

```cs
public static float ColorDifference(Color a, Color b)
```
Returns the magnitude of a vector whose components is the RGB difference between `a` and `b` (x is the red difference, y is the green difference and z is the blue difference).

```cs
public static Vector3 MiddlePoint(Vector3[] inputs)
```
This is UNUSED, but remains functional. Returns a vector that is all `input` added together / the length of `inputs`.

```cs
public static int CheckIfMore(int value, int[] values)
```
This is UNUSED, but remains functional. Returns the amount of `values` element that is less than `value`.

```cs
public static float GetSqrDistance(Vector3 a, Vector3 b)
public static float GetSqrDistance(Vector3 a, Vector3 b, bool ignorey)
```
Returns (a - b).sqrMagnitude. If `ignoreY` is true, `a` and `b` will have their y components set to 0 before doing the substraction.

On the overload without a `ignoreY` parameter, it acts as if the value was false.

```cs
public static float GetDistance(Vector3 a, Vector3 b)
public static float GetDistance(Vector2 a, Vector2 b)
public static float GetDistance(Vector3 a, Vector3 b, bool ignoreY)
```
Returns (a - b).magnitude. If `ignoreY` is true, `a` and `b` will have their y components set to 0 before doing the substraction.

On the first overload, the method behaves as if `ignoreY` was false.

The second overload is UNUSED, but remains functional. It does the same operations as the first overload, but there's no z component.

```cs
public static float GetDistance(float a, float b)
```
Returns the absolute value of a - b.

```cs
public static float LoopValue(float value, float min, float max)
public static float LoopValue(float value, float min, float max, bool floor)
```
This is UNUSED. Returns a value derived from `value` by doing the following on it:

- Add `max` as long as `value` is lower than `min`
- Set value to %= `max` as long as `value` is higher than `max`
- If `floor` is true, `value` gets floored
- `value` is returned

On the overload without a `floor` parameter, the value defaults to false.

```cs
public static Color RainbowColor(int variant)
public static Color RainbowColor(int variant, float colorspeed)
public static Color RainbowColor(int variant, float colorspeed, float min, float limit, float alpha)
```
Return a color with the following components (r, g and b are clamped from `min` to `limit`):

- r: 2.0 * Sin((Time.time + `variant` + 20.0) * `colorspeed`)
- g: 2.0 * Sin((Time.time + `variant`) * `colorspeed`)
- b: 2.0 * Sin((Time.time + `variant` + 60.0) * `colorspeed`)
- w: `alpha`

Intuitively, `variant` is a way to make nearby colors using this method to be out of phase with each other by phase shifting the rgb, `colorspeed` is the period of the rgb, `min` is the lowest value of rgb (this should be at least 0.0 for consistency) and `limit` is the highest value of rgb (this should be at most 1.0 for consistency).

NOTE: Because of the * 2.0 followed by a clamp, these sin curves don't features their full range. Assuming a clamp from 0.0 to 1.0, it means that 1/2 of the time, the value will get mapped to 0.0 and for 1/4 of the time, it will map to 1.0 leaving only 1/4 of the time being values between 0.0 and 1.0.

For the first 2 overloads, here are the default values of the parameters not sent:

- `colorspeed`: 5.9
- `min`: 0.0
- `limit`: 1.0
- `alpha`: 1.0

The second overload is UNUSED, but remains functional.

```cs
public static float ColorMagnitude(Color color)
```
Returns (`color`.r, `color`.g, `color`.b).magnitude.

```cs
public static float Snap(in float value, in float step)
```
If `step` is 0.0, `value` is returned, otherwise, the return value is `value` / `step` + 0.5 floored * `step`. The intuitive way to think about this method is it returns the nearest number from `value` that is divisible by `step`.

```cs
public static Vector3 Snap(in Vector3 value, in Vector3 step)
```
Returns a vector where each component is Snap(`value`.x/y/z, `step`.x/y/z) with matching components for both vectors.

```cs
public static IEnumerator LateAngle(Transform obj, Vector3 angle, bool local, WaitForSeconds time)
```
Yield for `time` then change the angles of `obj` to `angle`. If `local` is true, the field of `obj` set is localEulerAngles instead of eulerAngles.

```cs
public static bool CheckAllBool(bool[] array, int[] values, bool state)
public static bool CheckAllBool(bool[] array, bool state)
```
Returns true if all `values` elements contains an `array` index whose matching value is `state`, false otherwise. All `values` elements must contain valid `array` indexes or an exception will likely be thrown.

For the overload without a `values` parameter, the method checks all of `array` elements to be `state` instead. It behaves the same then if `values` contained all the valid `array` indexes.

```cs
public static float GetDistance01(float start, float end, float multiplier)
```
This is UNUSED, but remains functional. Returns 1.0 - GetDistance(`start`, `end`) * `multiplier` clamped from 0.0 to 1.0.

```cs
public static float GetPercentage(float start, float end, float currentvalue)
```
Returns 1.0 - (`end` - `currentvalue`) / (`end` - `start` clamped from 0.001 to infiniy). The intuitive way to think about this is it returns the progression ratio of `currentvalue` assuming a starting point of `start` and a goal of `end`. It is assumed that `start` <= `currentvalue` <= `end`.

```cs
public static IEnumerator GradualScale(Transform obj, Vector3 target, float frametime, bool destroy)
```
Changes `obj` scale towards `target` over `frametime` + 1.0 amount of frames. If `destroy` is true, `obj` gets destroyed once this is over.

```cs
public static void SetParenting(Transform t, Transform parent)
```
Child `t` to `parent` and does the following on `t` after:

- Reset local position to Vector3.zero
- Reset scale to Vector3.one
- Reset angles to Vector3.zero

```cs
public static IEnumerator LateParent(Transform a, Transform b, float time)
```
This is UNUSED, but remains functional. Waits `time` amount of seconds before childing `a` to `b`.

```cs
public static IEnumerator Shrink(Transform obj, float frametime, bool deleteonend)
```
Changes `obj`'s scale towards Vector3.zero over `frametime` + 1.0 amount of frames. If `deleteonend` is true, `obj` is destroyed once this is over.

```cs
public static void FaceTowardsY(Transform t, Vector3 pos)
```
Make `t` LookAt `pos` whitout changing the x and z angles.

```cs
public static void Insert(int value, ref int[] array)
```
Sets `array` to a new array with one more element at the end with a value of `value`.

```cs
public static void ChangeLayer(Transform obj, int layer)
```
Changes the layer of all children of `obj` to `layer`.

```cs
public static bool ArrayIsEmpty(object[] input)
public static bool ArrayIsEmpty(object[] input, bool stringcheck)
```
Returns true if every elements in `input` is either null or its ToString returns `null`, false otherwise. If `stringcheck` is false, only the null value check is done, not the ToString.

The overload without the `stringcheck` parameter behaves as if its value was false.

```cs
public static Vector3 RandomItemBounce(float range, float height)
```
Returns a vector where the y component is `height` and the x/z components comes the vector returned by ClampMagnitude(RandomVector(`range` / 2, `range` / 2), `range` / 2, `range` / 2). Intuitively, it's a vector with a random x/z direction (each random between -`range` / 2 and `range` / 2) whose magnitude is `range` / 2 and whose y component is `height`.

```cs
public static IEnumerator DelayedPosition(Transform obj, Vector3 pos, float delay, bool local)
```
Waits `delay` amount of seconds (or a frame if it's 0.0 or below) then set `obj` position to `pos` (or local position instead if `local` is true).

```cs
public static IEnumerator ArcMovement(GameObject obj, Vector3 targetpos, float height, float frametime)
public static IEnumerator ArcMovement(GameObject obj, Vector3 startpos, Vector3 targetpos, Vector3 spin, float height, float frametime, bool destroyonend)
```
Move `obj` as long as it isn't null from `startpos` to `targetpos` using a BeizierCurve3 with a `ymax` of `height` while rotating it by `spin` * TieFramerate(1.0) on each frames over the course of `frametime` amount of frames and set `obj` position to `targetpos` once the movement is complete. If `destroyonend` is true, instead of setting `obj` position after the movement, `obj` gets destroyed. At the end of the coroutine, a frame is yielded. If `obj` is null during any frames of its movement, the coroutine abruptly ends.

For the second overload, the `spin` parameter defaults to Vector3.zero and the `destroyonend` parameter defaults to false. The overload only starts the other coroutine and yields a frame.

```cs
public static int[] OrganizeArrayInt(int[] inputarray, int[] order)
```
Returns a new array where every elements is `order` in the same order they appear, but every elements that doesn't also exists in `inputarray` are removed.

```cs
public static Vector3 MultiplyVector(Vector3 a, Vector3 b)
```
Returns a vector where each components is calculated by multiplying the matching components of `a` and `b`.

```cs
public static int[] GradualFill(int ammount)
public static int[] GradualFill(int startat, int ammount)
```
Returns an array with length `ammount` where the first element is `startat` and each subsequent elements increases by 1.

The overload without a `startat` parameter has its value default to 0.

```cs
public static Vector3 ChildScale(Vector3 scale, Transform parent, bool swapZY)
```
Returns a vector where each components is calculated by dividing the matching components of `scale` over `parent`. If `swapZY` is true, only the x component is calculated with matching components (the y component is `scale`.y / `parent`.z and the z component is `scale`.z / `parent`.y).

```cs
public static Vector3 CardinalSnap(Vector3 angle)
public static Vector3 CardinalSnap(Vector3 angle, bool cameratied)
```
Returns a normalized vector with a direction that is the closest of the 4 cardinal directions of `angle`.y. If `cameratied` is true, `globalcamdir`.forward is added to the result and then normalized. More precisely, here's the logic to determine the base direction of the vector based on `angle`.y:

- Between 45.0 and 135.0 exclusive: Vector3.left
- At least 135.0 and strictly lower than 225.0: Vector3.forward
- At least 225.0 and strictly lower than 315.0: Vector3.right
- At least 315.0 or strictly lower than 45.0: Vector3.back

For the overload without a `cameratied` parameter, the value defaults to false.

```cs
public static Vector3 CardinalSnap(Transform obj)
public static Vector3 CardinalSnap(Transform obj, int directions)
public static Vector3 CardinalSnap(Vector3 obj, int directions)
```
Return `obj` where the y component of the vector is Mathf.RoundToInt(`obj`.y / (360.0 / `directions`)) * (360.0 / `directions`). Intuitively, this y component will be the closest delimiter number among the ones that are obtained by splitting 360.0 in `directions` amount of equal length sections. It means it's meant to work with euler angles with an arbitrary amount of cardinal directions (a cardinal direction here is one evenly split segments of 360.0).

The first 2 overloads are UNUSED, but remains functional.

For the overloads taking `obj` as a transform instead of a vector, the value becomes `obj`.tansform.eulerAngles.

For the overload not taking a `directions` parameter, the value defaults to 4.

```cs
public static float CardinalSnap(float angle)
public static float CardinalSnap(float angle, int directions)
```
Return Mathf.RoundToInt(`angle` / (360 / `directions`)) * (360 / `directions`). Intuitively, this returns the closest delimiter number among the ones that are obtained by splitting 360 in `directions` amount of equal length sections. It means it's meant to work with euler angles with an arbitrary amount of cardinal directions (a cardinal direction here is one evenly split segments of 360). NOTE: The 360 / `direction` division is incorrect because it's an integer division, NOT a float one so there can be loss of precisions if `directions` isn't a whole divisor of 360 resulting in uneven segments splits.

For the overload without a `directions` parameter, the value defaults to 4.

```cs
public static Vector3 CardinalSnap8(Vector3 angle, bool cameratied)
```
Returns a normalized vector with a direction that is the closest of the 8 cardinal directions of `angle`.y. If `cameratied` is true, `globalcamdir`.forward is added to the result and then normalized. More precisely, here's the logic to determine the base direction of the vector based on `angle`.y (this is the value before the vector gets normalized):

- Between 22.5 and 67.5 exclusive: (-1.0, 0.0, -1.0)
- At least 67.5 and strictly lower than 112.5: (-1.0, 0.0, 0.0)
- At least 112.5 and strictly lower than 157.5: (-1.0, 0.0, 1.0)
- At least 157.5 and strictly lower than 202.5: (0.0, 0.0, 1.0)
- At least 202.5 and strictly lower than 247.5: (1.0, 0.0, 1.0)
- At least 247.5 and strictly lower than 292.5: (1.0, 0.0, 0.0)
- At least 292.5 and strictly lower than 337.5: (1.0, 0.0, -1.0)
- At least 337.5 or strictly lower than 22.5: (0.0, 0.0, -1.0)

```cs
public static float ClampToMinMax(float v, float min, float max)
public static float ClampToMinMax(float v, float min, float max, bool lowest)
```
Returns `min` or `max` depending on `v` and Lerp(`min`, `max`, 0.5) which is the midpoint of the range. If `v` is lower than the midpoint, `max` is returned, `min` otherwise. If `lowest` is true, this logic is inverted so `min` is returned if `v` is lower than the midpoint, `max` otherwise.

On the overload without a `lowest` parameter, the value defaults to false.

```cs
public static bool InBetween(float v, float a, float b)
```
Return true if `v` is higher or equal than `a` and strictly lower than `b`, false otherwise.

```cs
public static void LookAt(Transform obj, Vector3 targetp)
public static void LookAt(Transform obj, Vector3 targetp, bool keepangle)
```
Have `obj` LookAt `targetp` and set `obj` x and z angles to 0.0 after. This effectively changes `obj` y angles to the one aligned with `targetp`. If `keepangle` is true, the x and z angles of `obj` won't change instead of being set to 0.0.

On the overload without a `keepangle` parameter, it behaves as if the value was false.

```cs
public static float[] Divisions(int divisions)
```
Return an array of length `divisions` where the first element is 1.0 / (`divisions` + 1) and each subsequent elements increases by 1.0 / (`divisions` + 1). Intuitively, it creates a sequence of increasing fractions where the numerator goes from 1.0 to `divisions` - 1 and the denominator is `divisions` + 1.

```cs
public static IEnumerator LatePos(Transform obj, Vector3 pos, float delay, bool keeptrying)
```
Wait `delay` amount of seconds before setting `obj` position to `pos` if `obj` isn't null by then. If `keeptrying` is true, it changes the behavior where the coroutine will set `obj` position to `pos` if it isn't null on each frames for the next `delay` amount of frames.

```cs
public static Vector2 GetDirection(in Vector2 a, in Vector2 b)
public static Vector3 GetDirection(in Vector3 a, in Vector3 b)
public static Vector3 GetDirection(in Vector3 a, in Vector3 b, bool ignoreY)
```
Returns (`a` - `b`).normalized. If `ignoreY` is true, the vectors used to do the subtraction are `a` and `b` where their y component is 0.0.

For the overloads without the `ignoreY` parameter, the value defaults to false.

```cs
public static Vector3 GetDirection4(Vector3 a, Vector3 b, bool ignoreY)
```
Returns the same value than GetDirection(`a`, `b`, true). The `ignoreY` parameter doesn't semantically change anything.

```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 target, float frametime, bool smooth, bool local)
```
Changes `obj` position (or local position if `local` is true) towards `target` via a Lerp (or SmoothLerp if `smooth` is true) over the course of `frametime` amount of frames.

```cs
public static Vector3 GlobalScale(Vector3 desiredscale, Vector3 parentscale, bool flipZY)
```
This is UNUSED, but remails functional. Returns a vector where each components is calculated by dividing the matching components of `desiredscale` over `parentscale`. If `flipZY` is true, only the x component is calculated with matching components (the y component is `desiredscale`.z / `parentscale`.z and the z component is `desiredscale`.y / `parentscale`.y).

```cs
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t)
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t, float onlythrough, float onlyafter)
```
Return a vector obtained by performing a Mathf.SmoothLerp on each matching components of `a` and `b` using `t` as the factor. If `onlythrough` is above 0.0 and `t` is less than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. If `onlyafter` is above 0.0 and `t` is greater than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. 

For the first overload, the `onlythrough` and `onlyafter` parameters defaults to 0.0.

```cs
public static IEnumerator TempIgnoreCollision(Collider a, Collider b, float seconds)
```
Ignores all collisions between `a` and `b` for `seconds` amount of seconds before uniginoring all collisions between `a` and `b`.

```cs
public static IEnumerator FlipSpriteBool(SpriteRenderer sprite, bool x, bool y, float everyxframes, float duringxframes)
```
A coroutine that will causes `sprite` to have their flipX toggled if `x` is true and their flipY toggled if `y` is true every `everyxframes` amount of frames for a total duration of `duringxframes`. If `duringxframes` is -1, the duration is infinite and the coroutine runs indefintely until it is manually stopped.

```cs
public static Vector3 VectorFromString(string[] inputs)
```
Returns a vector where the x component is `inputs[0]`, the y component is `inputs[1]` and the z component is `inputs[2]` after converting each strings to float.

```cs
public static IEnumerator ForceFailsafe(Transform obj, Vector3 target, float cooldown)
```
This is UNUSED, but remains functional. If after waiting for `cooldown` amount of frames, the GetDistance between `obj` position and `target` is greater than 0.45000002, the following happens (the coroutine does nothing useful otherwise):

- DeathSmoke(`obj`.position) called
- `obj`.position set to `target`
- DeathSmoke(`target`) called

## GameObject creation and destruction

```cs
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite, int sortorder)
```
Creates and return a new GameObject that has the following properties:

- Childed to `parent` or `GUICamera` if `parent` is null
- Layer 5 (`UI`)
- No angles
- Local position of `pos`
- Scale of `size`
- If `sprite` isn't null, the GameObject will have a SpriteRenderer with a sprite of `sprite` and a sortingOrder of `sortorder`

On the first 2 overloads, here are what the parameters defaults to:

- `size`: Vector3.one
- `sprite`: null
- `sortorder`: 0

```cs
public static IEnumerator DelayedObj(float delay, string path, Vector3 position, string sound, float destroy)
```
After waiting for `delay` amount of seconds, load a GameObject using `path` ressources path placing it at `position`. If `destroy` is above 0.0, the GameObject will be destroyed in `destroy` amount of seconds.

```cs
public static void DestroyAllChildren(Transform parent)
```
Destroys all children GameObject of `parent`.

```cs
public static SpriteRenderer NewSolidColor(string name, Color color)
public static SpriteRenderer NewSolidColor(string name, Color color, float pixelsperunit, Vector3 position, Vector2 pivot)
```
Returns the SpriteRenderer of a newly created GameObject named `name` positioned at `position` where the SpriteRenderer was created by applying a 1x1 Texture2D on the entire sprite whose sole pixel color is `color`, the pivot of the sprite is `pivot` and the pixelsPerUnit of the sprite is `pixelsperunit`.

On the first overload, the `pixelsperunit` defaults to 100.0, the `position` defaults to Vector3.zero and the `pivot` defaults to (0.5, 0.5).

```cs
public static Transform Create9Box(Vector3 position, Vector2 size, int type, int sortorder, Color color, bool grow)
```
Returns the transform of a newly created GameObject that is configured as a UI object using a 9 box configured sprites. More precisely, it's configured like the following:

- Name of `9Box`
- Childed to the `GUICamera`
- Layer 5 (`UI`)
- Local position of `position`
- No Angles
- If `grow` is false, the scale is Vector3.one. If it's true, it's Vector3.zero, but with a DialogueAnim component added which will bring the scale to Vector3.one over some frames
- A SpriteRenderer is added with the following properties:
    - sprite: Loaded from `Sprites/GUI/9Box/boxX` where `X` is `type`
    - color: `color`
    - drawMode: Tiled
    - tileMode: Adaptive

```cs
public static GameObject WaterSplash(Vector3 pos, Vector3 size)
```
Instantiates and returns a GameObject using the `Prefabs/Objects/WaterSplash` prefab positioned at `pos` with a scale of `size`. The GameObject will be destroyed in 0.75 seconds.

```cs
public static IEnumerator LightingBolt(Vector3 a, Vector3 b, int segments, float variant, Color color, float frametime, float lineduration)
```
Instantiates a `Prefabs/Particles/LightingBolt` prefab initialy positioned at `a` with some configuration of its TrailRenderer:

- material.color: `color`
- colorGradient: A gradient that has 2 keyframes each with a color of `color`
- time: `lineduration`

The GameObject will progressively move towards `b` over the course of around `frametime` amount of frames in `segments` amount of movement segments (clamped from 2 so there's always at least 2 segments). These movement segments are evenly distributed in time and positioning over the course of moving from `a` to `b` via a Lerp. On any movement sections except the first and last one, there will be a random offset applied to the target of the movement determined by the return of RandomVector(`variant`). The intuitive way to think about this is that this will result in the trail looking jagged as a random offset gets applied to each target of each movement, but the starting position is always correct so it will result in a random zig zag pattern.

Once the whole movement is completed, the GameObject will be destroyed in 1.0 seconds and a frame is yielded.

```cs
public static void ScreamWaves(Vector3 pos)
```
Starts a Waves coroutine with the following parameters:

- pos: `pos`
- ammount: 5
- frametime: 20.0
- delay: 0.25 seconds
- invert: false
- tridimentional: false
- color: null

```cs
public static IEnumerator Waves(Vector3 pos, int ammount, float frametime, WaitForSeconds delay, bool invert, bool tridimentional, Color? color)
```
Instantiates the `Prefabs/Objects/SphereGlowEffect` prefab `ammount` times all positioned at `pos` each with a Renderer's material.color of `color` if `color` isn't null with a wait of `delay` between each instantiation. The scale of each prefabs is done via a GradualScale with a frametime of `frametime` and a destroy of true so each instances gets destroyed after the scaling has completed. As for the initial and target scale:

- If `invert` is false, the initial scale is Vector3.zero and the target is (50.0, 50.0, 1.0), but the z component is 50.0 instead if `tridimentional` is true
- If `invert` is true, the initial scale and target are reversed from the above

```cs
public static SpriteRenderer NewSpriteObject(Vector3 position, Transform parent, Sprite sprite)
public static SpriteRenderer NewSpriteObject(string name, Vector3 position, Vector3 rotation, Transform parent, Sprite sprite, Material mat)
```
Returns the SpriteRenderer of a newly created GameObject with the following properties:

- name of `name`
- Childed to `parent`
- Layer 14 (`Sprite`)
- Local position of `position`
- Angles of `rotation`
- sprite of `sprite`
- material of `mat`

For the first overload, the `name` defaults to `tempsprite`, `rotation` defaults to Vector3.zero and `mat` defaults to the `spritemat`.

```cs
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale)
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale, bool gui)
public static GameObject Create6Wheel(Color[] colors, Sprite[] sprites, Vector3 spritescale, bool gui)
```
Returns a newly created GameObject named `Wheel` showing a 6 segments wheel each with a sprite rendered over a colored `guisprites[122]` (???). Here are what the parameters configures:

- `colors`: The material.color of each segment's `guisprites[122]` background sprite
- `sprites`: The foreground sprite rendered in each segment over their background sprite
- `spritescale`: The scale of of all foreground sprites
- `gui`: If this is true, all sprites renders on layer 5 (`UI`) and the GameObject gets childed to the `GUICamera` via SetParenting instead of being rooted to the scene

The first 2 overloads are UNUSED, but remains functional and will have the `colors` value default to 6 elements all being Color.white and the `gui` value default to false.

```cs
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset)
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset, bool UI)
```
Instantiates `Prefabs/Particles/Heal` in a new GameObject with the following properties which will be destroyed in 3.0 seconds:

- Childed to `parent`
- Scale of `size`
- Local position of `offset`
- If `UI` is true, the layer is set to 5 (`UI`)

The overload without an `UI` parameter will have a default value of false.

```cs
public static void CreateCursor(Transform parent)
```
Set `cursor` to a newly created GameObject with the following properties:

- Name of `Cursor`
- Childed to `parent`
- Layer is 5 (`UI`)
- Scale of Vector3.one
- Local position of Vector3.zero
- Has a SpriteRenderer with a sprite of `cursorsprite[0]` and a sortingOrder of 1
- Has a SpriteBounce. MessageBounce is called on it after adding it

```cs
public static void DestroyTemp(GameObject obj, float time)
public static void DestroyTemp(ref GameObject obj, float time)
```
Set `obj` position to (0.0, -9999.0, 0.0) and destroy it in `time` amount of seconds. If `obj` is passed by ref, it is set to null after the Destroy call.

The method does nothing if `obj` is null.

```cs
public static Transform NewChild(string name, Transform parent)
```
Create and return the transform of a newly created GameObject childed to `parent` named `name` with angles and local position of Vector3.zero and a scale of Vector3.one.

```cs
public static GameObject CreateRock(Vector3 pos, Vector3 size, Vector3 rotation)
```
Returns a newly instantiated GameObject using the `Prefabs/Objects/CrackedRock` prefab with the following properties:

- Position: `pos`
- Angles: `rotation`
- Scale: `size`
- The MeshCollider is destroyed
- The Fader is destroyed

```cs
public static void CrackRock(Transform parent, bool destroyparent)
```
Instantiates `Prefabs/Objects/CrackRockBreak` with the same position, angles and scale of `parent`. If `destroyparent` is true, `parent` gets destroyed after the instantiation.

## Inputs

```cs
private static bool AnyJoyKey(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
private static bool AnyJoyStick()
```
TODO: Involves InputIO logic that's not documented yet

```cs
private void GetJoystick()
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static bool KeyHold(Directions direction)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static void ResetKeyHold()
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static bool GetKey(int id)
public static bool GetKey(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static int MultipleKeys(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static string[] Controllers()
```
Return Input.GetJoystickNames(), but each element that has a length of 1 or lower are removed.

```cs
public static bool AnyKeyButThis(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet

## Particles and visual effects

```cs
public static void RefreshWind(Renderer t)
```
Set the following float shader properties of the material of `t`:

- `_ShakeDisplacement`: Random between `map`.`windspeed` / 2.0 and `map`.`windspeed`. NOTE: This property is unused in the wind shader so this does nothing, the intended property was likely `_ShakeWindspeed`
- `_ShakeBending`: Random between `map`.`windintensity` / 2.0 and `map`.`windintensity`
- `_ShakeDisplacement`: Random between 0.075 and 0.25

```cs
public static void HurtParticle(Vector3 pos, bool playsound)
```
Calls HitPart(`pos`) and if `playsound` is true, this if followed by a call to PlaySound(`Hurt`).

```cs
public static GameObject PlayParticle(string name, Vector3 position)
public static GameObject PlayParticle(string name, Vector3 position, float time)
public static GameObject PlayParticle(string name, string sound, Vector3 position)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime, int rendersort)
```
Returns a newly created GameObject that is instantiated from the prefab at `Prefabs/Particles/X` where `X` is `name` at position `position` with angles of `angle` and whose Renderer has a material.renderQueue of `rendersort`. If `sound` isn't null, PlaySound(`sound`) is called before creating the GameObject. If `alivetime` is higher than 0.0, the GameObject will get destroyed in `alivetime` amount of seconds.

For the overloads without all parameters, their values defaults to the following:

- `sound`: null
- `angle`: (-90.0, 0.0, 0.0)
- `alivetime`: 5.0 (or the `time` value for the second overload)
- `rendersort`: 3000

```cs
public static IEnumerator ChapterName(int chapterid)
```
Plays the animations and effects for a chapter start cutscene where the chapter number is `chapterid` + 1. NOTE: Only values of 0 through 6 are valid `chapterid` values.

```cs
public static IEnumerator ShakeObject(Transform obj, Vector3 shake, float frametime, bool returntostart)
```
For `frametime` + 1.0 amount of frames, `obj` position will be offset by RandomVector(`shake`) each frames. Once this is done, if `returntostart` is true, `obj` position is reset to its value at the start of this coroutine.

```cs
public static void SortLights()
public static void SortLights(MeshRenderer[] r)
```
For every `r` element that isn't null and has a material.shader set to `fakelight`, their material.renderQueue is set to 2500 + `MainCamera`.WorldToViewportPoint(the `r` element).z * 100.0 (or * 10.0 if in a `battle`). If `r` is null, this process will be done on all MeshRenderer present on the scene that satisfies the conditions.

The parameterless overload has a `r` value of null.

Intuitively, this method allows to have all `fakelight` materials in `r` (or the whole scene) be visually sorted due to the renderQueue.

```cs
public static void DeathSmoke(Vector3 pos)
public static void DeathSmoke(Vector3 pos, Vector3 size)
public static void DeathSmoke(Vector3 pos, Vector3 size, int render)
```
Does the following to emit some `deathpart`:

- Set `deathpart` scale to `size`
- Set `deathpart` position to `pos`
- Emit(20) called on `deathpart`
- Set `deathpart`'s Renderer's material.renderQueue to `render`

For the first 2 overloads, the `size` value defaults to Vector3.one and the `render` value defaults to 3000.

```cs
public static void HitPart(Vector3 pos)
```
Set `hitpart` position to `pos` and call Play on it to play its particles.

```cs
public static void DisableRender(Renderer render, float tolerance, Vector3 offset)
```
Changes `render`.shadowCastingMode depending on a CheckIfCamera test on `render` position using a tolerance of `tolerance` and an offset of `offset`. If the test returns true, the new shadowCastingMode is On, ShadowsOnly otherwise.

```cs
public static void DisableRender(GameObject render, float tolerance, Vector3 offset)
```
This is UNUSED, but remains functional. Set the enablement of `render` depending on the return of CheckIfCamera on `render` position using a tolerance of `tolerance` and an offset of `offset`.

```cs
public static void UpdateOutlines()
```
Calls Shader.SetGlobalFloat to set `GlobalOutline` to a value that depends on `map`.`areaid` and the `enableoutline` setting. Here's the exact logic used:

- If `enableoutline` is 0 (OFF), the value is set to 0.0
- If `enableoutline` is 1 (LOW), the value is set to 0.1
- If `enableoutline` is 2 (HIGH), it depends if `map`.`areaid` is `MetalLake` or not:
    - If it is, the value is set to 0.1
    - If it is not, the value is set to 0.3

## Special procedures

```cs
public static void EndMiniGame(bool antialias, int score)
```
Performs the necessary operations to end a Termacade game where the FXAA component's enablement is set to `antialias` where the game ended with a score of `score`. Among the tasks done by this method includes calling SetRenderTexture(0), calling GetRewardTokens(`score`).

The `antialias` parameter is expected to come from the component containing the Termacade game logic and it is expected to contain the initial enablement of the FXAA component when the game starts. This is used to restore the enablement to its original state. The `score` is used to determine how much game tokens should be awarded by GetRewardTokens.

```cs
public static void UpdateArea(int newarea)
```
Performs the tasks needed to process an area change which happens if needed on a MapControl's Start. More precisely, the following is done:

- If flag 596 is true (talked to the Roach Elder in the postgame) and `areaid` is 15 (meaning this is changing the area from the Giant's Lair), flag 597 is set to true (the `elder` entity on GiantLairSaplingPlains has a `limit` on it so this will cause his disappearence from that map).
- All regional flags are reset
- `areaid` is updated to `newarea`
- The matching `librarystuff[4]` area flag of `newarea` is set to true which marks it as visible on the world map

```cs
public static void UpdateShops()
```
Set all `avaliablebadgepool` to each be a RandomSort version of their matching `badgeshops`. This happens on MapControl's Start, NPCControl's SetBadgePool(true) for a medals shop and during event 22 (loading a save file) when upgrading the save file to 1.2.0.

Additionally, for `avaliablebadgepool[0]` (Merab) specifically, if flag 587 is true (bought all Merab's medals), but `badgeshops[0]` isn't empty, flag 587 is reset to false. This corrects an inconsistency which can happen on a freshly upgraded 1.2.0 save file where the flag saying all medals were bought is true, but there's still some to be bought after the upgrade.

```cs
public static void SetMonitor()
```
This is an empty method, it does nothing, but remains used by FlappyBee under normal gameplay.

```cs
public static void LoadMap()
public static void LoadMap(int id)
public static void LoadMap(int id, bool recreateplayers)
```
Decomission and destroy the current `map` before loading a new one with a Maps id of `id`. If `recreateplayers` is true, all `playerdata` gets destroyed followed by setting `player` to null before this process.

This method is documented in detail in the map loading documentation.

```cs
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout)
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout, Vector3? camn, Vector3? camp)
```
Performs the cutscene animations of the party sleeping and getting healed after. Here are what the parameters do:

- `caller`: If the value is not null, their `entity`.`talking` will be set to false before the animation and their `entity`.`emoticoncooldown` will be reset to 0.0 after the animation
- `position`: If the value is not null, the `player` position to set during the cutscene while the screen is all black
- `changemusic`: If true, a ChangeMusic call will happen at the end of the cutscene where the musicclip is `music[0].clip.name` or null if `music[0]`.clip is null
- `nofadeout`: If true, there will not be a fadeout transition at the end of the cutscene
- `camn`: If `position` and this value are not null, `map`.`camlimitneg` will be set to this value during the cutscene
- `camp`: If `position` and this value are not null, `map`.`camlimitpos` will be set to this value during the cutscene

For the second overload, `camn` and `camp` values defaults to null. All the overload does is start a coroutine of the other overload followed by a frame yield.

This coroutine is meant to be stored in `chaptername` as it is set to null at the end of the coroutine so this field can be used to track its progress. The cutscene involves these major steps:

- PlaySound(`Inn`, 5) called
- FadeMusic(0.1) called
- PlayTransition(4, 9999, 0.01, Color.black) called
- All frames will be yielded for up to 700.0 frames until the transition is over
- If `position` is not null, the `player` is repositioned and the camera limits adjusted if needed with a yield of 0.1 seconds and a TeleportFollowers(true) call
- Yield all frames for up to 60.0 frames as long as `sounds[5]` is still playing
- If `nofadeout` is false: PlayTransition(1, 0, 0.15, Color.clear). 0.5 seconds will be yielded after the call
- Heal called

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 targetpos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos, NPCControl caller)
```
Changes the current map with a loading transition. Check the map loading documentation to learn more.

The first overload is UNUSED, but remains functional.

## Library

```cs
public static bool[] GetLibraryBools(int page)
```
Returns a new array whose elements is `librarystuff[page]` meaning the 256 flags array matching the `page`.

```cs
public static bool HasRecipe(int id)
```
This is UNUSED, but remains functional. Returns the flag value matching the recipe entry in `librarystuff` where `id` is the resulting item in the recipe. If no such recipe exists, false is always returned.

```cs
public static int GetRecipeID(int id)
```
Returns the recipe id from `libraryorder` where the resulting item id is `id` or -1 if no such recipe exists.

```cs
public static void UpdateJounal()
public static void UpdateJounal(Library type, int variable)
```
If `type` is Bestiary and the game is `inbattle`, `librarystuff[1, variable]` is set to true which mark the bestiary entry as obtained.

Otherwise, the following happens to mark a `librarystuff` flag as obtained if applicable as well as revealing in the HUD the obtention of a flag:

- If `variable` isn't negative, `librarystuff[type, variable]` is set to true which marks the entry as obtained
- `discoveryhud` is set to 350.0 which reveals the HUD element
- The `Disc` animation clip plays on the child of `discoverymessage` or the `Arch` clip if `type` is Logbook
- The 2nd, 3rd and 4th child of `discoverymessage` have their enablement changed depending on `type`:
    - Logbook: Only the 4th child is enabled, 2nd and 3rd are disabled
    - Other `type` values: Only the 3rd child is enabled, 2nd and 4th are disabled

The parameterless overload has the `type` value default to Discovery and `variable` to -1. This has the effect of not doing any changes to `librarystuff`, but still reveal `discoveryhud`.

```cs
private static int[] GetLibraryIDs(int index)
```
Get the array of effectively used `libraryorder[index]` (meaning the length is `librarylimit[index]`). Intuitively, it returns an array of all valid `librarystuff[index]` id so it acts as a Library enum value.

```cs
private static int[] GetLibraryOrder(int id)
```
Returns all effectively used (meaning length is `librarylimit[id]`) entry ids in `libraryorder` where the first dimension index is `id` (so it acts as a Library enum value).

```cs
public static int GetEnemyPortrait(int id)
```
Obtains the `librarysprites` index of the portrait sprite associated with the enemy whose id is `id`. There's some special cases logic to this which is documented in the GetEnemyPortrait documentation.

## Save file

```cs
public static string SaveFile(Vector3? savepos)
```
Returns a save file compliant text from the current state of MainManager. Check the save file format documentation to learn more.

```cs
public static void ReloadSave()
```
This method is a helper to setup an event 22 start (loading a save file). Here are the steps performed:

- StopSound(`CrowdChatter`) called
- FadeMusic(0.1) called
- Stop all coroutines of `battle` and destroy it if it exists
- Stop all coroutines of `events`
- Destroy all `playerdata` entities's GameObject
- Set RenderSettings.skybox to null
- Destroy the `map`
- `events`.StartEvent called to start event 22 without a caller

```cs
public static LoadData? Load(int file, bool lite)
```
Completely load the save file with filename `saveX.dat` where `X` is `file` from the game's root directory with various error handling. In practice, only the values 0, 1 and 2 are valid `file` values. For information on the file format itself, check the save file format documentation. The return value is a struct that contains some metadata about the save used to present them in the StartMenu or null if the loading failed. If `lite` is true, the save will be loaded to gather these metadata, but it won't result in any changes to any MainManager fields.

This method handles a lot of potential errors, here is a more detailed list of their logic:

- If the save file doesn't exist, null is returned
- Reading the save file is enclosed in a try catch block that will return null if the reading of the file failed
- The entire method is enclosed in a try catch block that will handle any exceptions by a Debug.Log call and returning null
- In `lite` loading, the reading of the save file's filename is enclosed in a try catch block that will handle exceptions by setting the LoadData's filename to `|color,1|SLOT X - NO FILE NAME` where `X` is `file` + 1

Additionally, in non `lite` loading, the following additional steps are done in this loading process:

- ChangeParty without fromscratch is called with the party composition in the save file once their information have been read sucessfully
- ApplyBadges is called right before returning on a sucessful load.

```cs
public static int SaveProgressIcons()
```
Returns the amount of progress icons that should be rendered on the StartMenu. Each count if a specific flag slot is true, here's the list:

- 41 (end of chapter 1)
- 88 (end of chapter 2)
- 299 (end of chapter 3)
- 345 (end of chapter 4)
- 347 (end of chapter 6)
- 346 (got the flame brooch)
- 555 (post game)

```cs
public static bool Save(Vector3? savepos)
```
Calls InputIO.Save(`savepos`) which will persist the game state to the current save file. Check the save file format documentation to learn more.

## Items

```cs
public static int MixIngredients(int item1, int item2)
```
Search `recipedata` for a compatible recipe where `item1` and `item2` matches the ingredients and returns the result of the matching recipe if one is found or 8 (`Mistake`) if no such recipe exists. To search of a single item recipe, `item2` needs to be -1. The method supports having `item1` and `item2` being reversed compared to the recipe for a double item recipe since it will normalise the order before searching.

If `item1` or `item2` is 156 (`Guarana`), no search is performed and instead, the result is a random result determined among the following:

- If `item2` isn't 156 (`Guarana`):
    - 1/13 to be `RoastBerry`
    - 2/13 to be `BigMistake`
    - 1/13 to be `BerryJuice`
    - 2/13 to be `BerrySmoothie`
    - 1/13 to be `Mistake`
    - 3/13 to be `CookedJellyBean`
    - 1/13 to be `TangyJam`
    - 1/13 to be `SpicyBomb`
    - 1/13 to be `BurlyBomb`
- Otherwise (`item1` is 156, `Guarana`):
    - 2/11 to be `RoastBerry`
    - 4/11 to be `Mistake`
    - 2/11 to be `BigMistake`
    - 1/11 to be `BerryJuice`
    - 1/11 to be `CookedJellyBean`
    - 1/11 to be `BerrySmoothie`

```cs
public static string GetItemString(bool emptyinv)
```
Returns a string that represents a `,` separated list of all item id in `items[0]` (standard items inventory). If `emptyinv` is true, `items[0]` is reset to a new empty list before returning the string which empty empties the entire standard items inventory.

```cs
public static void InventoryFromString(string inp, int itemtype, bool addd)
```
Replace `items[itemtype]` with the list of item ids obtained by splitting `inp` by `,`. If `addd` is true, the items will be added to the existing list instead.

NOTE: `inp` must be a a `,` separated list of valid item ids and `itemtype` must be 0, 1 or 2 or an exception will be thrown.

```cs
public static Sprite GetItemSprite(bool badge, int id)
```
Returns `itemsprites[0, id]` if `badge` is false or `itemsprites[1, id]` if `badge` is true.

```cs
public static int HowManyItem(int type, int id)
```
Returns the number of occurences of the item whose id is `id` in `items[type]`.

```cs
private static void GetRewardTokens(int score)
```
Changes the value of flagvar 1 (amount of game tokens) depending on `score`, `battleresult` (which indicate if the minigame was beaten) and flagvar 6 (indicates the minigame played, 0 is FlappyBee and 1 is MazeGame). The value is always clamped from 2 to 999999 at the end.

More specifically, this is the logic used to determine the value (before the clamping):

- If `battleresult` is false (the minigame was failed), the value is `score` / 200.0 floored
- Otherwise, if flagvar 6 is 0 (minigame was FlappyBee), the value is `score` / 95.0 floored
- Otherwise (minigame is assumed to be MazeGame), the value is `score` / 140.0 floored

```cs
public static ItemUse GetItemUse(int id)
public static ItemUse GetItemUse(int id, int itemtype)
```
Returns the ItemUse of an item with id `id`. Check the ItemUse documentation for more details. NOTE: `itemtype` must be 0 or an exception will be thrown.

The overload without an `itemtype` parameter has its value default to 0.

```cs
public static int? GetItemHeal(int id)
```
This is UNUSED. it involved the unused BattleControl.TryHealEnemyItem coroutine.

```cs
public static int DoItemEffect(ItemUsage type, int value, int? characterid)
```
Process an ItemUsage with value `value` on `playerdata[characterid]` if `characterid` isn't null. Check the DoItemEffect documentation to learn more.

## General game utilities

```cs
public static Maps CurrentMap()
```
Returns `map`.`mapid`.

```cs
public static bool IsPaused()
```
Returns true if any of the following conditions is true (false is returned otherwise):

- The game is in a `minipause`
- The game is in a `pause`
- The game is `inevent`
- A `battle` is in progress and it is `inevent`

```cs
public static bool CheckIfCanExist(int[] requires, int[] limit, int regionalflag)
```
Returns false if all of the following conditions are satisfied which indicates that the conditions sent are satisfied (true otherwise indicating the conditions are NOT satisifed):

- Any flag slots that matches the absolute value of any `limit` elements of -2 or lower must be false
- If `requires` is considered not empty, all flags slots specified in `requires` must be true. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `limit` is considered not empty, all flags slots specified in `limit` must be false. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `regionalflag` applies, the regionalflag slot `regionalflag` must be false. The condition applies only if `regionalflag` isn't negative and `map` isn't null (meaning the map isn't being loaded)

This method implements the backing logic of a ConditionChecker and NPCControl's existence logic. NOTE: The return value is inverted: false if conditions are satisifed, true if conditions are NOT satisfied.

```cs
public static float TieFramerate(float value)
```
Returns `value` * Time.smoothDeltaTime * 60.0. This is intended to be used to count game frames or to scale events happening over time. A `value` of 1.0 returns the actual smoothed out frametime while a `value` lower than 1.0 returns a fraction of it.

NOTE: The multiplication by 60.0 implies that a return value of 1.0 means that exactly 1/60th of a second passed since the last couple of frames on average. It strongly assumes that the target FPS is 60 so anything higher will result in the value being lower. The game was NOT designed to handle this and will result in a lot of events that happens over time being much slower than normal due to this strong FPS dependency.

```cs
public static bool FrameDifference(int frames)
```
Returns true if Time.frameCount % the game's framerate / 60.0 ceiled is 0. The `frames` parameter does nothing.

In practice, this will always return true with vsync set to OFF. If vsync is set to ON, the amount of time true is returned varies and it depends on the monitor's refresh rate:

- 60hz and lower: true is always returned
- Between 60hz and 120hz exclusive: true is returned every 2 frames, false otherwise
- Between 120hz and 180hz exclusive: true is returned every 3 frames, false otherwise
- etc... (keep adding 60hz to the bounds to decrease the frequency at which true is returned)

```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller, string soundatend)
```
The following happens to move `obj` from `start` to `end`:

- If `startdelay` is above 0.0, wait for `startdelay` amount of seconds
- Changes `obj` position from `start` to `end` via a Lerp (or SmoothLerp if `smooth` is true) over the course of `frametime` + 1.0 amount of frames
- `obj` position is set to `end`
- If `soundatend` isn't null, PlaySound(`soundatend`) is called
- A frame is yielded
- If `shrink` is above 0.0, a DialogueAnim is added to `obj` with a `shrink` of true and a `shrinkspeed` of `shrink`
- If `destroytime` isn't negative, `obj` is destroyed in `destroytime` amount of seconds (or immediately if the value is 0.0)
- `caller` is invoked with a parameter of false

The first 2 overloads will start a third overload coroutine and yield a frame.

For the overloads without a `soundatend` parameter, the value defaults to null.

The second overload is UNUSED, but remains functional.

For the first overload, here are the default values for the parameters not sent:

- `startdelay`: 0.0
- `shrink`: 0.0
- `destroytime`: -1.0

```cs
public static int CrystalBerryAmmount()
```
Returns the amount of `crystalbflags` that are true.
