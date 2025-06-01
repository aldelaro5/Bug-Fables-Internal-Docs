# Particles and visual effects
These methods instantiates particles or plays other visual effects.

## Particles
These methods deals with ParticleSystem specifically.

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
public static void HurtParticle(Vector3 pos, bool playsound)
```
Calls HitPart(`pos`) and if `playsound` is true, this if followed by a call to PlaySound(`Hurt`).

```cs
public static void HitPart(Vector3 pos)
```
Set `hitpart` position to `pos` and call Play on it to play its particles.

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

## Other visual effects
These methods deals with other visual effects than ParticleSystem.

```cs
public static void RefreshWind(Renderer t)
```
Set the following float shader properties of the material of `t`:

- `_ShakeDisplacement`: Random between `map`.`windspeed` / 2.0 and `map`.`windspeed`. NOTE: This property is unused in the wind shader so this does nothing, the intended property was likely `_ShakeWindspeed`
- `_ShakeBending`: Random between `map`.`windintensity` / 2.0 and `map`.`windintensity`
- `_ShakeDisplacement`: Random between 0.075 and 0.25

For more information, consult the [MapControl graphics configuration](../../MapControl/Graphics%20configuration.md).

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
public static void DisableRender(Renderer render, float tolerance, Vector3 offset)
```
Changes `render`.shadowCastingMode depending on a CheckIfCamera test on `render` position using a tolerance of `tolerance` and an offset of `offset`. If the test returns true, the new shadowCastingMode is On, ShadowsOnly otherwise.

```cs
public static void UpdateOutlines()
```
Calls Shader.SetGlobalFloat to set `GlobalOutline` to a value that depends on `map`.`areaid` and the `enableoutline` setting. Here's the exact logic used:

- If `enableoutline` is 0 (OFF), the value is set to 0.0
- If `enableoutline` is 1 (LOW), the value is set to 0.1
- If `enableoutline` is 2 (HIGH), it depends if `map`.`areaid` is `MetalLake` or not:
    - If it is, the value is set to 0.1
    - If it is not, the value is set to 0.3

## Unused or broken
These methods are unused or not functional.

```cs
public static void DisableRender(GameObject render, float tolerance, Vector3 offset)
```
This is UNUSED, but remains functional. Set the enablement of `render` depending on the return of CheckIfCamera on `render` position using a tolerance of `tolerance` and an offset of `offset`.
