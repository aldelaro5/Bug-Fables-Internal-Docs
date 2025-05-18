# Transition
Transitions are visual effects happening over time to hide loading processes done by the game such as map loading, battle loading or just to indicate a story transition such as what happens in the InnSleep animation.

There are 6 available transitions in the game each defined by their hardcoded id. They are organised in pairs as they are meant to be used together, but it's possible to mix and match some of them:

- 0: Fade in to a color
- 1: Fade out
- 2: `leafsprites` moving in with a color
- 3: `leafsprites` moving out
- 4: Iris wipe in to a color
- 5: Iris wipe out from a color

An in transition is meant to be used first and later on, an out transition is meant to be used to restore the visuals and teardown the transition objects. 

## Transition coroutine
All transitions ends up being processed by the following coroutine:

```cs
private static IEnumerator Transition(int id, int data, float speed, Color color)
```
`id` is the id of the transition, `data` is an optional parameter that is specific to the transition (some may not do anything with it), `speed` modifies how fast the transition will happen and `color` determines on some transitions what color they will use to go into or from.

The way these transition works is they use an array field called instance.`transitionobj` which typically will features SpriteRenderers presented with a very high sortingOrder on layer 5 (`UI`) childed to the `GUICamera`. This results in the sprites having very high rendering priority and effectively can cover the whole screen since it's on the `GUICamera` so there's no depth.

Here's what the coroutine does at a high level:

- instance.`intransition` is set to true
- The transition with id `id` is executed
- instance.`intransition` is set to false
- Yield for a frame
- `transition` is set to null

This coroutine return is meant to be stored in `transition` since it's set to null at the end so the caller can use this to track if it's in progress.

## PlayTransition and other transition players
Normally, the coroutine isn't started directly. Instead, the PlayTransition method is meant to be called by the game to start any transitions:

```cs
public static void PlayTransition(int id, int data, float speed, Color color)
```
The method will stop the `transition` coroutine if it was ongoing and set it to a new Transition coroutine call with the following parameters:

- id: `id`
- data: `data`
- speed: `speed`
- color: `color`, but if each RGB component was higher than 0.95, the color value will be (0.95, 0.95, 0.95, `color`.a) which is almost pure white

Additionally, if `transitionobj` isn't empty and `id` is 0, 2, 4 or 5 (so it's an in transition or an iris wipe out transition), all non null `transitionobj` elements gets destroyed before the Transition call.

There's also other methods that ends up calling PlayTransition in a certain way. Here's all of them:

```cs
public static void FadeIn()
public static void FadeIn(float speed)
public static void FadeIn(float speed, Color color)
```
Calls PlayTransition(0, 0, `speed`, `color`).

On the overload without a `speed` parameter, the value defaults to 0.1.

On the overloads without a `color` parameter, the value defaults to Color.black.

```cs
public static void FadeOut()
public static void FadeOut(float speed)
```
Calls PlayTransition(1, 0, `speed`, Color.black).

On the overload without a `speed` parameter, the value defaults to 0.1.

```cs
public static SpriteRenderer Dimmer(int fadetype)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed, Color color)
```
Calls PlayTransition(`fadetype`, 0, `fadespeed`, `color`) and then return `transitionobj[0]`'s SpriteRenderer.

For first 2 overloads, `fadespeed` will default to 0.1 and `color` defaults to pure black.

## GetTransitionSprite
Once PlayTransition was called, it's possible to obtain the SpriteRenderer of any `transitionobj` with the following method:

```cs
public static SpriteRenderer GetTransitionSprite()
public static SpriteRenderer GetTransitionSprite(int id)
```
Returns the SpriteRenderer of `transitionobj[id]`.

For the overload without an `id` parameter, the value defaults to 0.

## Transition execution
This section talks about the details of each transitions as they happen during the Transition coroutine.

### 0 - Fade in to a color
Fades the screen into the color `color`. The `speed` modifies how fast the lerp to `color` will go.

This transition does not use the `data` so any value will be ignored.

Only `transitionobj[0]` is used and it is the transform of a newly created GameObject named `Dimer` with a SpriteRenderer and it has the following properties:

- Childed to `GUICamera`
- Layer 5 (`UI`)
- Local position of 10.0 in z
- No angles
- sprite of the full rectangle of a newly created 1 pixel Texture2D with a color of `color` and a scale of (3000.0, 3000.0, 1.0)
- sortingOrder of 9999
- color of Color.clear

Once the GameObject is created, the transition is executed. It is done by changing `transitionobj[0]`'s SpriteRenderer's color towards `color` until the alpha is 1.0 or higher for up tp 600.0 frames. Each frame, the color is set to a lerp from the existing one to `color` with a factor of `speed` * `framestep`.

### 1 - Fade out
This transition undo what the fade in did by clearing the color and destroying the GameObject.

`data` and `color` are ignored, but the `speed` controls the transition in a similar way to the fade in.

Since this is assumed to be after a fade in, only `transitionobj[0]` is used. If it's not present, this transition does nothing.

The transition is executed changes `transitionobj[0]`'s SpriteRenderer's color towards Color.clear until the alpha is 0.0 or lower for up tp 600.0 frames. Each frame, the color is set to a lerp from the existing one to Color.clear with a factor of `speed` * `framestep`.

After, `transitionobj[0]` gets destroyed.

### 2 - `leafsprites` moving in with a color
A transition to cover the screen with `leafsprites` in various patterns available using the color `color`. The `speed` indicates the fraction of TieFramerate to use as the lerp factor on each frame of the transition.

The `data` value is the `leafsprites` index to use for the sprites that will cover the screen. Valid values ranges from 0 to 8. Notably, value 4 is for hexagon sprites which have a slightly different transition logic than other values.

`transitionobj` will contain one element per `leafsprites` displayed as they get created with a SpriteRenderer. For hexagon sprites, it's always hardcoded to 10 elements and for other sprites, the length is `leafpos`'s length. The target local positions in either cases are hardcoded. For non hexagon, it's from `Data/LeafPos` and for hexagons, they are hardccoded inside the method. Here are the hardcoded target local positions for hexagons:

- (-6.3, 3.7, 10.0)
- (0.0, 3.7, 10.0)
- (6.3, 3.7, 10.0)
- (-9.4, -1.8, 10.0)
- (-3.125, -1.8, 10.0)
- (3.125, -1.8, 10.0)
- (9.4, -1.8, 10.0)
- (-6.3, -7.2, 10.0)
- (0.0, -7.2, 10.0)
- (6.3, -7.2, 10.0)

Besides the target local positions, each `transitionobj` features the following property upon creation:

- Name of `LeafX` where `X` is the index of the sprite starting from 0
- Childed to `GUICamera`
- Layer 5 (`UI`)
- The scale, angles and local position are determined based on if it's hexagon sprites or not:
    - hexagon: scale of (0.75, 0.75, 0.75), no angles and a local position of the position offset + (0.0, 10.0 + 10.0 * `X`) where `X` is the index of the sprite starting from 0
    - non hexagon: scale of Vector3.one, angles of (0.0, 0.0, random between 0.0 and 360.0) and a local position of (-20.0, random between -1.0 and 1.0, 10.0)

After all `transitionobj` are created, the transition is executed with the following:

- For hexagon sprites: Each `transitionobj` local position changes to its target via a lerp from the existing one with a factor of TieFramerate(`speed`). This happens each frames until `transitionobj[0]` local position becomes exactly (`leafpos[0]`.x, `leafpos[0]`.y, 10.0). NOTE: This stopping condition is incorrect and will result in the transition never ending (in practice, this will be fixed when the out transition is started since it will destroy all `transitionobj`)
- For non hexagon sprites:  Each `transitionobj` local position changes to its target via a lerp from the existing one with a factor of TieFramerate(`speed`). This happens each frames until `transitionobj[0]` local position becomes exactly (`leafpos[0]`.x, `leafpos[0]`.y, 10.0). Additionally, on each frame, the z angle angles increases by 0.05 * GetSqrDistance between the existing local position and the target local position (using a z of 10.0).

### 3 - `leafsprites` moving out
This transition undo what the `leafsprites` moving in transition did by removing all the `transitionobj` sprites on the screen and destroying them. The `speed` indicates the fraction of TieFramerate to use as the lerp factor on each frame of the transition. This transition does nothing if `transitionobj` is empty or null.

The `data` value should always be the same as the one used for the battle in transition as supplying a different value may lead to inconsistencies.

Here is how this transition is executed:

- `data` is 4 (hexagon sprites): Each `transitionobj` local position changes to its original target position (they are hardcoded and documented above in the battle in transition) + (0.0, -20 - 10 * `X`) where `X` is the `transitionobj` index. The movement is done via a lerp from the existing local position with a factor of TieFramerate(`speed`). This is done until the distance between `transitionobj[0]`.localPosition and (20.0, 10.0, 0.0) is 0.1 or lower. NOTE: This stopping condition is incorrect and will result in the transition never truly ending which means the `transitionobj` will never get destroyed and remain rendered off camera
- `data` is not 4 (not hexagon sprites): Each `transitionobj` local position changes to (20.0, 10.0, 0.0) via a lerp from the existing local position with a factor of TieFramerate(`speed`). Additionally on each frame, the z angle increases by 0.02 * GetSqrDistance between the existing local position and (20.0, 10.0, 0.0). This is done until the distance between `transitionobj[0]`.localPosition and (20.0, 10.0, 0.0) is 0.1 or lower.

After the transition, all `transitionobj` gets destroyed.

### 4 - Iris wipe in to a color
Fades the screen into the color `color` as the screen closes in in a circle that constrict itself until the entire screen is covered. The `speed` indicates the fraction of TieFramerate to use as the lerp factor on each frame of the transition. This transition is followed by a fade out.

The `data` represents the sortingOrder to use on the sprite of the transition.

Only `transitionobj[0]` is used and it is the transform of a newly created GameObject instantiated from the `Prefabs/Objects/RoundTransition` (which is already on layer 5 meaning `UI`) and it has the following properties:

- Childed to `GUICamera`
- Local position of 10.0 in z
- No angles
- sortingOrder of `data`

Once the GameObject is created, the transition is executed. It is done by changing `transitionobj[0]`'s SpriteRenderer's material.color towards `color` until the alpha is 0.985 or higher. Each frame, the color is set to a lerp from the existing one to `color` with a factor of TieFramerate(`speed`). Also on each frame, the `_Cutoff` shader property on the material is set to 1.0 - material.color.apha. This is done because the texture on the prefab is a grayscale gradient in a circle shape so by decreasing the alpha cutoff, it progressively reveals the color in an iris closing fashion.

Once this is over, `transitionobj[0]` gets destroyed in 0.2 seconds.

From there, a fade out transition immediately happens. The logic is the same as described above (including the reassignement of `transitionobj[0]`), but with the following differences:

- The `speed` value is always 1.0
- The sortingOrder is `data`

### 5 - Iris wipe out from a color
This transition does the reverse of the iris wipe in transition from `color` as the screen reveals itself with a growing circle until the entire screen is revealed. The `speed` indicates the fraction of TieFramerate to use as the lerp factor on each frame of the transition, but it is divided by 2.0 before usage.

The `data` represents the sortingOrder to use on the sprite of the transition.

Only `transitionobj[0]` is used and it is the transform of a newly created GameObject instantiated from the `Prefabs/Objects/RoundTransition` (which is already on layer 5 meaning `UI`) and it has the following properties:

- Childed to `GUICamera`
- Local position of 10.0 in z
- No angles
- color of `color`
- sortingOrder of `data`

Once the GameObject is created, the transition is executed. It is done by changing `transitionobj[0]`'s SpriteRenderer's material.color towards Color.clear until the alpha is 0.05 or lower. Each frame, the color is set to a lerp from the existing one to Color.clear with a factor of TieFramerate(`speed`). Also on each frame, the `_Cutoff` shader property on the material is set to 1.0 - material.color.apha. This is done because the texture on the prefab is a grayscale gradient in a circle shape so by increasing the alpha cutoff, it progressively hides the color in an iris opening fashion.

Once this is over, `transitionobj[0]` gets destroyed in 0.2 seconds.
