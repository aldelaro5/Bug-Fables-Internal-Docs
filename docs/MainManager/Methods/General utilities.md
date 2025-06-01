# General utilities
These methods are made to be called in a wide variety of situations.

## Math and algorhythms
These methods are utilities that operates on primitives types or collections of primitives and perform operations on them.

```cs
public static bool IsWithin(int value, int min, int max)
```
Returns true if `value` is both greater than or equal `min` and less than or equal `max`, false otherwise.

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
public static bool CheckAllBool(bool[] array, bool state)
public static bool CheckAllBool(bool[] array, int[] values, bool state)
```
Returns true if all `values` elements contains an `array` index whose matching value is `state`, false otherwise. All `values` elements must contain valid `array` indexes or an exception will likely be thrown.

For the overload without a `values` parameter, the method checks all of `array` elements to be `state` instead. It behaves the same then if `values` contained all the valid `array` indexes.

```cs
public static float GetPercentage(float start, float end, float currentvalue)
```
Returns 1.0 - (`end` - `currentvalue`) / (`end` - `start` clamped from 0.001 to infiniy). The intuitive way to think about this is it returns the progression ratio of `currentvalue` assuming a starting point of `start` and a goal of `end`. It is assumed that `start` <= `currentvalue` <= `end`.

```cs
public static void Insert(int value, ref int[] array)
```
Sets `array` to a new array with one more element at the end with a value of `value`.

```cs
public static bool ArrayIsEmpty(object[] input)
public static bool ArrayIsEmpty(object[] input, bool stringcheck)
```
Returns true if every elements in `input` is either null or its ToString returns `null`, false otherwise. If `stringcheck` is false, only the null value check is done, not the ToString.

The overload without the `stringcheck` parameter behaves as if its value was false.

```cs
public static int[] OrganizeArrayInt(int[] inputarray, int[] order)
```
Returns a new array where every elements is `order` in the same order they appear, but every elements that doesn't also exists in `inputarray` are removed.

```cs
public static int[] GradualFill(int ammount)
public static int[] GradualFill(int startat, int ammount)
```
Returns an array with length `ammount` where the first element is `startat` and each subsequent elements increases by 1.

The overload without a `startat` parameter has its value default to 0.

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
public static float[] Divisions(int divisions)
```
Return an array of length `divisions` where the first element is 1.0 / (`divisions` + 1) and each subsequent elements increases by 1.0 / (`divisions` + 1). Intuitively, it creates a sequence of increasing fractions where the numerator goes from 1.0 to `divisions` - 1 and the denominator is `divisions` + 1.

```cs
public static float GetDistance(float a, float b)
```
Returns the absolute value of a - b.

## Vector math
These methods perform math operation on Vector2 or Vector3.

```cs
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limits)
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limitspos, Vector3 limitsneg)
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
public static Vector3 MultiplyVector(Vector3 a, Vector3 b)
```
Returns a vector where each components is calculated by multiplying the matching components of `a` and `b`.

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
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t)
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t, float onlythrough, float onlyafter)
```
Return a vector obtained by performing a Mathf.SmoothLerp on each matching components of `a` and `b` using `t` as the factor. If `onlythrough` is above 0.0 and `t` is less than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. If `onlyafter` is above 0.0 and `t` is greater than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. 

For the first overload, the `onlythrough` and `onlyafter` parameters defaults to 0.0.

```cs
public static Vector3 RandomItemBounce(float range, float height)
```
Returns a vector where the y component is `height` and the x/z components comes the vector returned by ClampMagnitude(RandomVector(`range` / 2, `range` / 2), `range` / 2, `range` / 2). Intuitively, it's a vector with a random x/z direction (each random between -`range` / 2 and `range` / 2) whose magnitude is `range` / 2 and whose y component is `height`.

```cs
public static Vector3 VectorFromString(string[] inputs)
```
Returns a vector where the x component is `inputs[0]`, the y component is `inputs[1]` and the z component is `inputs[2]` after converting each strings to float.

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

## GameObject utilities
These methods perform operations on GameObject, Transforms or common GameObject components.

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
public static IEnumerator LerpObject(Transform obj, Vector3 position, float speed, bool destroyonend)
```
Moves `obj` via a Lerp from its current position to `position` with a factor that grows from 0.0 to 1.0 at a rate of TieFramerate(`speed`) each frame. Once the Lerp completed, `obj` gets destroyed if `destroyend` is true.

```cs
public static IEnumerator Spin(Transform obj, Vector3 target, float frametime, bool smooth)
```
Changes `obj` angles towards `target` over the course of `frametime` + 1.0 amount of frames via a Lerp. If `smooth` is true, a SmoothLerp is used instead.

```cs
public static void SetParenting(Transform t, Transform parent)
```
Child `t` to `parent` and does the following on `t` after:

- Reset local position to Vector3.zero
- Reset scale to Vector3.one
- Reset angles to Vector3.zero

```cs
public static IEnumerator Shrink(Transform obj, float frametime, bool deleteonend)
```
Changes `obj`'s scale towards Vector3.zero over `frametime` + 1.0 amount of frames. If `deleteonend` is true, `obj` is destroyed once this is over.

```cs
public static void FaceTowardsY(Transform t, Vector3 pos)
```
Make `t` LookAt `pos` whitout changing the x and z angles.

```cs
public static void ChangeLayer(Transform obj, int layer)
```
Changes the layer of all children of `obj` to `layer`.

```cs
public static Vector3 ChildScale(Vector3 scale, Transform parent, bool swapZY)
```
Returns a vector where each components is calculated by dividing the matching components of `scale` over `parent`.localScale. If `swapZY` is true, only the x component is calculated with matching components (the y component is `scale`.y / `parent`.localScale.z and the z component is `scale`.z / `parent`.localScale.y).

```cs
public static void LookAt(Transform obj, Vector3 targetp)
public static void LookAt(Transform obj, Vector3 targetp, bool keepangle)
```
Have `obj` LookAt `targetp` and set `obj` x and z angles to 0.0 after. This effectively changes `obj` y angles to the one aligned with `targetp`. If `keepangle` is true, the x and z angles of `obj` won't change instead of being set to 0.0.

On the overload without a `keepangle` parameter, it behaves as if the value was false.

```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 target, float frametime, bool smooth, bool local)
```
Changes `obj` position (or local position if `local` is true) towards `target` via a Lerp (or SmoothLerp if `smooth` is true) over the course of `frametime` amount of frames.

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
public static IEnumerator TempIgnoreCollision(Collider a, Collider b, float seconds)
```
Ignores all collisions between `a` and `b` for `seconds` amount of seconds before uniginoring all collisions between `a` and `b`.

```cs
public static IEnumerator GradualScale(Transform obj, Vector3 target, float frametime, bool destroy)
```
Changes `obj` scale towards `target` over `frametime` + 1.0 amount of frames. If `destroy` is true, `obj` gets destroyed once this is over.

```cs
public static IEnumerator FlipSpriteBool(SpriteRenderer sprite, bool x, bool y, float everyxframes, float duringxframes)
```
A coroutine that will causes `sprite` to have their flipX toggled if `x` is true and their flipY toggled if `y` is true every `everyxframes` amount of frames for a total duration of `duringxframes`. If `duringxframes` is -1, the duration is infinite and the coroutine runs indefintely until it is manually stopped.

## Defered operations
These methods perform various operations, but only after waiting a certain amount of time.

```cs
public static IEnumerator DelayedPosition(Transform obj, Vector3 pos, float delay, bool local)
```
Yields `delay` amount of seconds (or a frame if it's 0.0 or below) then set `obj` position to `pos` (or local position instead if `local` is true).

```cs
public static IEnumerator LatePos(Transform obj, Vector3 pos, float delay, bool keeptrying)
```
Yields `delay` amount of seconds before setting `obj` position to `pos` if `obj` isn't null by then. If `keeptrying` is true, it changes the behavior where the coroutine will set `obj` position to `pos` if it isn't null on each frames for the next `delay` amount of frames.

```cs
public static IEnumerator LateAngle(Transform obj, Vector3 angle, bool local, WaitForSeconds time)
```
Yield for `time` then change the angles of `obj` to `angle`. If `local` is true, the field of `obj` set is localEulerAngles instead of eulerAngles.

## Color utilities
These methods performan various operations on colors.

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
public static float ColorDifference(Color a, Color b)
```
Returns the magnitude of a vector whose components is the RGB difference between `a` and `b` (x is the red difference, y is the green difference and z is the blue difference).

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

## Snapping
These methods takes some values or vectors and reduces them to a single value or vector that is the closest among a predetermined set of values or vectors.

```cs
public static float Snap(in float value, in float step)
```
If `step` is 0.0, `value` is returned, otherwise, the return value is `value` / `step` + 0.5 floored * `step`. The intuitive way to think about this method is it returns the nearest number from `value` that is divisible by `step`.

```cs
public static Vector3 Snap(in Vector3 value, in Vector3 step)
```
Returns a vector where each component is Snap(`value`.x/y/z, `step`.x/y/z) with matching components for both vectors.

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

## Curves
These methods returns the value along a curve which is useful for changing values through time.

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
public static float QuadraticYUnclamped(float currentx, float startx, float endx, float modifier)
```
Returns the y value of a quadratic curve expressed like (x - `endx`)(x - `start`) * a where x is `currentx` and a is 0.0 - `modifier`. Intuitively, `startx` and `endx` represents the 2 zeroes of the quadratic curve, `currentx` represents the x of the point we want to get its y and `modifier` is a scaler to the curve. To have an unscaled quadratic curve, `modifier` needs to be -1.0.

```cs
public static IEnumerator ArcMovement(GameObject obj, Vector3 targetpos, float height, float frametime)
public static IEnumerator ArcMovement(GameObject obj, Vector3 startpos, Vector3 targetpos, Vector3 spin, float height, float frametime, bool destroyonend)
```
Move `obj` as long as it isn't null from `startpos` to `targetpos` using a BeizierCurve3 with a `ymax` of `height` while rotating it by `spin` * TieFramerate(1.0) on each frames over the course of `frametime` amount of frames and set `obj` position to `targetpos` once the movement is complete. If `destroyonend` is true, instead of setting `obj` position after the movement, `obj` gets destroyed. At the end of the coroutine, a frame is yielded. If `obj` is null during any frames of its movement, the coroutine abruptly ends.

For the second overload, the `spin` parameter defaults to Vector3.zero and the `destroyonend` parameter defaults to false. The overload only starts the other coroutine and yields a frame.

## Unused or broken
These methods are unused or are not functional.

```cs
public static IEnumerator ForceFailsafe(Transform obj, Vector3 target, float cooldown)
```
This is UNUSED, but remains functional. If after waiting for `cooldown` amount of frames, the GetDistance between `obj` position and `target` is greater than 0.45000002, the following happens (the coroutine does nothing useful otherwise):

- DeathSmoke(`obj`.position) called
- `obj`.position set to `target`
- DeathSmoke(`target`) called

```cs
public static Vector3 GlobalScale(Vector3 desiredscale, Vector3 parentscale, bool flipZY)
```
This is UNUSED, but remails functional. Returns a vector where each components is calculated by dividing the matching components of `desiredscale` over `parentscale`. If `flipZY` is true, only the x component is calculated with matching components (the y component is `desiredscale`.z / `parentscale`.z and the z component is `desiredscale`.y / `parentscale`.y).

```cs
public static IEnumerator LateParent(Transform a, Transform b, float time)
```
This is UNUSED, but remains functional. Waits `time` amount of seconds before childing `a` to `b`.

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
public static float DimishingReturns(float input, float decay)
```
This is UNUSED, but remains functionsl. Returns (`input` - 1.0) * `decay`.

```cs
public static Vector2 GetPressedDirection(bool hold)
```
This is UNUSED, but remains functional. Returns a normalized vector that represents the accumulation of all 4 cardinal directions that are being pressed with up/down and left/right separation. This results in the following logic to determine the components of the vector tested in order (the vector gets normalised after all of this):

- x: -1.0 for left, 1.0 for right, 0.0 for neither
- y: 1.0 for up, -1.0 for down, 0.0 for neither

```cs
public static Vector3 LerpVectorAngleSmooth(Vector3 input, Vector3 target, Vector3 current, float ammount)
```
This is UNUSED, but remains functional. Returns a vector where each of its component is the currentVelocity output of a SmoothDampAngle from `input` to `target` with a smoothTime of `ammount` using matching components for all vectors.

```cs
public static float QuadraticYSemiClamped(float currentx, float startx, float endx, float modifier)
```
This is UNUSED. Returns the y value of a quadratic curve expressed like (x - `endx`)(x - `start`) * a where x is `currentx` and a is 0.0 - (`modifier` + GetDistance(`startx`, `endx`) / 10.0) / GetDistance(`startx`, `endx`). Intuitively, `startx` and `endx` represents the 2 zeroes of the quadratic curve and `currentx` represents the x of the point we want to get its y. `modifier` influences the scaler of the curve, but the math doesn't make sense.

```cs
public static Vector3 MiddlePoint(Vector3[] inputs)
```
This is UNUSED, but remains functional. Returns a vector that is all `input` added together / the length of `inputs`.

```cs
public static int CheckIfMore(int value, int[] values)
```
This is UNUSED, but remains functional. Returns the amount of `values` element that is less than `value`.

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
public static float GetDistance01(float start, float end, float multiplier)
```
This is UNUSED, but remains functional. Returns 1.0 - GetDistance(`start`, `end`) * `multiplier` clamped from 0.0 to 1.0.
