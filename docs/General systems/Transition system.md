
## Transition

```cs
public static SpriteRenderer Dimmer(int fadetype)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed, Color color)
```
Calls PlayTransition(`fadetype`, 0, `fadespeed`, `color`) and then return `transitionobj[0]`'s SpriteRenderer. For the other 2 overloads, `fadespeed` will default to 0.1 and `color` defaults to pure black.

More concretely, this starts the transition TODO: explain when transitions are properly documented

```cs
public static SpriteRenderer GetTransitionSprite()
public static SpriteRenderer GetTransitionSprite(int id)
```
Returns the SpriteRenderer of `transitionobj[id]`. For the overload, `id` defaults to 0.

```cs
public static void PlayTransition(int id, int data, float speed, Color color)
```
Stops `transition` if it was ongoing and set it to a new Transition coroutine call with the following parameters:

- id: `id`
- data: `data`
- speed: `speed`
- color: `color`, but if each RGB component was higher than 0.95, the color value will be (0.95, 0.95, 0.95, `color`.a) which is almost pure white

Additionally, if `transitionobj` isn't empty and `id` is 0, 2, 4 or 5 TODO: what's the pattern ??? , all non null `transitionobj` elements gets destroyed before the Transition call.

```cs
public static void FadeOut()
public static void FadeOut(float speed)
```
Calls PlayTransition(1, 0, `speed`, Color.black).

On the overload without a `speed` parameter, the value defaults to 0.1.

```cs
public static void FadeIn()
public static void FadeIn(float speed)
public static void FadeIn(float speed, Color color)
```
Calls PlayTransition(0, 0, `speed`, `color`).

On the overload without a `speed` parameter, the value defaults to 0.1.

On the overloads without a `color` parameter, the value defaults to Color.black.

```cs
private static IEnumerator Transition(int id, int data, float speed, Color color)
```
TODO: document the transition system separately since it gets complex
