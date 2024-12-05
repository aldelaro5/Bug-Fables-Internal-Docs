# ItemSpinAnim
A coroutine that can present a sprite icon with a spinning animation with an optional sound.

```cs
private IEnumerator ItemSpinAnim(Vector3 pos, Sprite sprite, bool playSound = true)
```

## Parameters

- `pos`: The position to show the spinning sprite
- `sprite`: The sprite to show with a spinning animation
- `playSound`: If true, the `Ding2` sound will play at 0.8 pitch

## Procedure

- If playsound is true, the `Ding2` sound will plays at 0.8 pitch
- A new sprite object is created named `effect` at pos using the sent sprite value on layer 15 (`3DUI`)
- Over the course of 21.0 frames:
    - The sprite's color is lerped from Color.Clear to pure white with 0.85 opacity
    - The sprite's scale is lerped from 2.0x to 1.0x
    - The sprite's z angle is lerped from 720.0 to 0.0
- Yield for 0.5 seconds
- Over the course of 21.0 frames the sprite's color is lerped from pure white with 0.85 opacity to Color.Clear
- The sprite object gets destroyed
