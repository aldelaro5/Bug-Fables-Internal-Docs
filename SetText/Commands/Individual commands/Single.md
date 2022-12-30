# Single

Toggle or set whether to render in [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)

## Syntax

(1)

````
|single|
````

(2)

````
|single,enable|
````

## Parameters

### `enable`:  `true` | `false`

Whether to enable [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) or to disable it. This must be equal to `true` or `false` without case sensitivity or an exception will be thrown.

## Remarks

(1) Toggles [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) from false to true or true to false depending on its existing state.
(2) Enables [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) if `enable` is true, disable it if it is `false`.

[Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) is a different text rendering method in SetText that involves rendering every line into their own letter slot rather than having one slot per letter. This is done to save a significant amount of letter slots. Visually, it mainly affects ` `'s gaps as they are no longer actual gap, but rather a ` ` that gets amended to the Text property of the TextMesh used to render the line. While the kerning is managed by Unity in this rendering method, the kerning looks very similar than in normal rendering.

### About line offset calculation

There is an issue where the offset to render in the current line is miscalculated compared to normal SetText rendering (it is much lower than it should be). This doesn't affect letter rendering since all the line is set in the `Text` property of the TextMesh, but it does affect other commands that needs this offset to be calculated properly. For example, many commands which renders sprites will not render the sprite at the correct position and the text will not get adequate spacing for the sprite to show correctly. This means this command should not be used when sprite rendering is performed later in the input string.

### About [FontEffects](../../Related%20Systems/FontEffects.md)

Since the [FontEffects](../../Related%20Systems/FontEffects.md) component was only designed to work on one letter per mesh, they do not work in [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).
