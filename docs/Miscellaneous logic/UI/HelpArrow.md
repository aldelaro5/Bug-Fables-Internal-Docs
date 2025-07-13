# HelpArrow
A component meant to be programmatically added to an existing GameObject by calling the NewArrow static method:

```cs
public static HelpArrow NewArrow(Transform parent, Vector3 offset, Color color, float distance, float size)
```
The method creates a GameObject named `Arrow` childed to `parent` with a local position of `offset` and this component attached. 

On Start, a SpriteRenderer is added using the `guisprites[196]` sprite (an arrow) using the MainManager.`spritedefaultunity` material on layer 15 (`3DUI`) initially disabled

The last 3 parameters of NewArrow configure the component's private fields:

- `color`: When satisfying the `distance` condition below, this is the color the arrow SpriteRenderer will go towards (more precisely, it's set to a lerp from pure white to this value with a factor of the absolute value of Sin(Time.time * 5.0))
- `distance`: Decides the maximum distance MainManager.`player` is allowed to be away from the `Arrow` before the SpriteRenderer gets disabled (it is enabled as long as this condition is fufilled)
- `size`: The scale of the arrow SpriteRendrer on Start is set to (0.45, 0.6, 1.0) * this value

There is also an exposed `lockarrow` field that when set to true will force the `distance` check to fail so the SpriteRenderer is always disabled.

Most of the logic happens on Update, but it won't be detailed here for the sake of brevity. Essentially, it is a compoennt specific to when MainManager.`player` is `Beetle` when near enough to a GameObject with this component to indicate the angle at which the object will go when using `Beetle`'s [Horn Slash](../../PlayerControl/Field%20abilities.md#horn-slash) ability on it. Specifically, it's used for:

- Hornable of type `IceCube` (used for [Dropplet](../../Entities/NPCControl/ObjectTypes/Dropplet.md))
- NPCControl of type [Enemy](../../Entities/NPCControl/Enemy.md) (`lockarrow` is set to true unless they are frozen)
- Any [PushRock](../../Entities/NPCControl/ObjectTypes/PushRock.md) with a `data[2]` that doesn't exist or is 0
