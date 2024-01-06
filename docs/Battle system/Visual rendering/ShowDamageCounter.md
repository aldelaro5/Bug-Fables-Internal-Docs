# ShowDamageCounter
This is a method that renders a visual counter of HP, TP or damage. This is only rendering the visual counter, it isn't mechanically involved in the damage pipeline, but it may be called as part of it.

```cs
private void ShowDamageCounter(int type, int ammount, Vector3 start, Vector3 end)
```

## Parameters

- `type`: The type of counter to show. Here are all the valid values:
    - 0: A damage counter
    - 1: An HP counter
    - 2: A TP counter
- `ammount`: The number to show inside the counter
- `start`: The starting position of the counter's lerp
- `end`: The end position of the counter's lerp

## Procedure

- A new GameObject is created named `X counter` where `X` is the ammount with a tag of `DelAftBtl` and a local scale of Vector3.zero
- Another GameObject is created named `sprite` childed to the counter just created with a SpriteRenderer on layer 15 (`3DUI`). The sprite is the `guisprites[counterspriteindex[X]]` where `X` is the type (4 for type 0, 40 for type 1 and 101 for type 2)
- For each `damcounters` whose position to the start is less than 1.5 away, start is increased by (1.25, 1.25, 1.25)
- The counter's position is set to start
- The sprite's local scale is set to Vector3.one / 1.5

From there, 2 [SetText](../../SetText/SetText.md) calls occurs in [non dialogue mode](../../SetText/Dialogue%20mode.md#non-dialogue-mode). The only distinguishing factors of the 2 calls is the input string and the position:

- The first has the input string be `|triui||halfline||center||sort,2||color,4|` followed by the ammount and it has a position of (0.0, 0.2, 0.0)
- The second has the input string be `|triui||halfline||center||sort,1|` followed by the ammount and it has a position of (0.08, 0.08, 0.08)

Both calls shares these properties:

- [fonttype](../../SetText/Notable%20states.md#fonttype) of 1 (`D3Streetism`)
- linebreak of 9999.0
- No tridimensional
- No cameraoffset
- size of Vector3.one * 1.75
- parent of the counter object
- No caller

The last step involves a CounterAnimation coroutine starting, but this coroutine now ends with this.

### CounterAnimation
This is a sub coroutine invoked as part of ShowDamageCounter. Here is its procedure (paraphrased a lot due to the presence of verbose rendering logic):

- The counter is added to `damcounters`
- The counter and the SpriteRenderer in its child have their layers set to 15 (`3DUI`)
- end is decreased by Vector3.back * (the amount of `damcounters` - 1) * 0.025 (this is done to avoid Z fighting between the `damcounters`)
- start is set to a new Vector3 with the following components:
    - X: The counter's x position
    - Y: The counter's y position clamped from -50.0 to 4.5
    - Z: The counter's z position - (the amount of `damcounters` - 1) * 0.025 (this is to avoid Z fighting between the `damcounters`)

The animation here depends on the type (for any type, the coroutine ends after by yielding a frame followed by the counter's destruction).

#### 0 (Damage counter)

- The SpriteRenderer Z angle is set to be a random angle between 0 and 360 exclusive (the X and Y angles are set to 0.0)
- Over the course of 10.0 frames:
    - The counter local scale is lerped from the existing one to (0.85, 1.15, 1.0) with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end with a factor of 3/10 of the game's frametime
    - The SpriteRenderer is rotated with  (0.0, 0.0, -0.7)
- Over the course of 10.0 frames:
    - The counter local scale is lerped from the existing one to Vector3.one with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end with a factor of 3/10 of the game's frametime
    - The SpriteRenderer is rotated with  (0.0, 0.0, -0.5)
- 0.3 seconds are yielded
- The counter is removed from `damcounters`
- Over the course of 20.0 frames:
    - The counter local scale is lerped from the existing one to Vector3.zero with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end + Vector3.up with a factor of 3/10 of the game's frametime
    - The SpriteRenderer is rotated with  (0.0, 0.0, -0.5)

#### 1 (HP counter)

- end is overriden to Vector3.up
- Over the course of 10.0 frames:
    - The counter local scale is lerped from the existing one to (0.65, 1.5, 1.0) with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end with a factor of 0.3
- Over the course of 10.0 frames:
    - The counter local scale is lerped from the existing one to (1.2, 0.9, 1.0) with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end with a factor of 0.3
- Over the course of 10.0 frames:
    - The counter local scale is lerped from the existing one to (1.0, 1.0, 1.0) with a factor of 1/5 of the game's frametime
    - The counter position is lerped from the existing one to start + end with a factor of 0.3
- 0.12 seconds are yielded
- The counter is removed from `damcounters`
- Over the course of 20.0 frames:
    - The counter local scale is lerped from the existing one to Vector3.zero with a factor of 1/10 of the game's frametime
    - The counter position is lerped from the existing one to start + end + Vector3.up * 2.0 with a factor of 1/5 of the game's frametime
    - The SpriteRenderer is rotated with  (0.0, 10.0, 0.0)

#### 2 (TP counter)
The same than the HP counter, but with the following changes:

- Before the first 10.0 frames lerp, a new UI object is created called `insidewheel` on layer 15 (`3DUI`) with the SpriteRenderer as the parent, local position of (0.0, 0.0, 0.01), size of Vector3.one, sortingOrder of the SpriteRenderer's - 1 using the sprite `guisprites[102]` (the rainbow wheel part of the TP counter)
- On each of the 10.0 frames lerp, the `insidewheel`'s Z angle is increased by 15x the game's frametime (the X and Y angles are always set to 0.0)
