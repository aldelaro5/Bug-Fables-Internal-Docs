# GameObject creation and destruction
These methods deals with creation or destroying GameObjects.

## Creation
These methods create GameObject in various ways.

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

## Destruction
These methods destroys GameObjects.

```cs
public static void DestroyAllChildren(Transform parent)
```
Destroys all children GameObject of `parent`.

```cs
public static void DestroyTemp(GameObject obj, float time)
public static void DestroyTemp(ref GameObject obj, float time)
```
Set `obj` position to (0.0, -9999.0, 0.0) and destroy it in `time` amount of seconds. If `obj` is passed by ref, it is set to null after the Destroy call.

The method does nothing if `obj` is null.
