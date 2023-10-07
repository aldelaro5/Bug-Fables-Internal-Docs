# Text advances

[SetText](../SetText.md) in [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) and [Dialogue mode](../Dialogue%20mode.md) has a normal yield time when a text skip isn't happening of 0.02 seconds or 0.03 seconds in `Japanese` for every standard letter. Every spaces do not get a wait and some punctuation has an additional fixed wait of 0.15 seconds. In [non dialogue mode](../Dialogue%20mode.md#non-dialogue-mode), an |[spd](../Individual%20commands/Spd.md),0| is prepended which already disables wait time and in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), wait times aren't supported at all. 

There are ways to skip the wait times on demand such as from the input string itself If the [speed](../Individual%20commands/Speed.md) is set to 0, but it is also possible to reduce it on demand by the player using either text skip system.

## Text skip

SetText allows to skip the wait time that would normally happen at the end of the [Dialogue post-processing](../Life%20Cycle.md#dialogue-post-processing) in [Dialogue mode](../Dialogue%20mode.md).

There are 2 instance fields of MainManager involved in this feature:

* `skiptext`: Tells if a text skip was requested by either pressing the confirm input or the cancel input without holding it. Initialized to false in [Dialogue setup](../Life%20Cycle.md#dialogue-setup) and overridden to false in multiple [Commands](../Commands/Commands.md) if applicable.
* `isholdingskip`: Tells if the cancel input is held which implies that `skiptext` is true as this is a super set of the `skiptext`. Initialized to false in [Dialogue setup](../Life%20Cycle.md#dialogue-setup)

When `skiptext` turns to true, SetText will process every letter without yielding until some conditions happens. For example, if a [next](../Individual%20commands/Next.md) command is processed, it will force the text skip to stop before yielding for the next textbox, but there are other [commands](../Commands/Commands.md) that stops it.

`isholdingskip` on the other hand is more aggressive: it will automatically pass confirmation inputs needed to release a [waitinput](../Notable%20states.md#waitinput) lock. This allows to pass textboxes as soon as the cooldown expires which for either skipping method is set to 16 frames. What this means is in the best case scenario, holding the Cancel input will still leave 16 frames of waiting between textboxes, but it will not require mashing.

## Override a text skip

SetText can force a textskip to end via [noskip](../Individual%20commands/Noskip.md) which ignores any request to text skip and will still upheld the normal yield time.
