# StatEffect
This coroutine will render a visual colored arrow to show some kind of effect done to the stats. This is purely for visual purposes and ephemeral: the arrow is destroyed shortly after.

```cs
private IEnumerator StatEffect(EntityControl target, int type)
```

## Parameters

- `target`: The entity to render the arrow on
- `type`: The type of effect to render which dictates the color, angles and local position to render the arrow

## Procedure
For the sake of brevety, the procedure won't be documented as it involves very verbose rendering logic. It mostly involves a SpriteBounce with some scaling lerps to make the arrow move on its own using non uniform scaling.

However, the type parameter is worth to document as it dictates how the arrow will be rendered:

- 0: CC00007F (80% red, half transparent) rendered vertically at the entity's `height` (the arrow points up)
- 1: 0000CC7F (80% blue, half transparent) rendered vertically at the entity's `height` (the arrow points up)
- 2: CC00007F (80% red, half transparent) rendered vertically at the entity's `height` + 2.0 with a 180.0 degrees rotation in z (the arrow points down)
- 3: 0000CC7F (80% blue, half transparent) rendered vertically at the entity's `height` + 2.0 with a 180.0 degrees rotation in z (the arrow points down)
- 4: 00CC007F (80% green, half transparent) rendered vertically at the entity's `height` (the arrow points up)
- 5: FFCC007F (mostly orange, half transparent) rendered vertically at the entity's `height` (the arrow points up)
