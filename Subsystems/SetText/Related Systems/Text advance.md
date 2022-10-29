# Text advances

The normal yield time when a text skip isn't happening is 0.02 seconds or 0.03 seconds in `Japanese` for every standard letter. Every spaces do not get a wait, but every punctuations (TODO: detail what a punctuation is) has a fixed wait of 0.15 seconds. If the [Speed](../Commands/Individual%20commands/Speed.md) is set to 0, it will also not wait for any letters and behave like a test skip.

## Text skip

SetText allows to skip the wait time that would normally happen at the end of the [life cycle > Dialogue post-processing](../life%20cycle.md#dialogue-post-processing) in [Dialogue mode](../Dialogue%20mode.md).

There are 2 instance fields of MainManager involved in this feature:

* `skiptext`: Tells if a text skip was requested by either pressing the confirm input or the cancel input without holding it. Initialized to false in [life cycle > Dialogue setup](../life%20cycle.md#dialogue-setup)
* `isholdingskip`: Tells if the cancel input is held which implies that `skiptext` is true as this is a super set of the `skiptext`. Initialized to false in [life cycle > Dialogue setup](../life%20cycle.md#dialogue-setup)

When `skiptext` turns to true, SetText will process every letter without yielding until some conditions happens. For example, if a [Next](../Commands/Individual%20commands/Next.md) command is processed, it will force the text skip to stop before yielding for the next textbox.

`isholdingskip` on the other hand is more aggressive: it will automatically pass confirmation inputs needed to release a [waitinput](../Global%20vars%20used/waitinput.md) lock. This allows to pass textboxes as soon as the cooldown expires which for either skipping method is set to 16 frames. What this means is in the best case scenario, holding the Cancel input will still leave 16 frames of waiting between textboxes, but it will not require mashing.

## Override a text skip

SetText can force a textskip to end via the [Noskip](../Commands/Individual%20commands/Noskip.md) MainManager static field which is done with some commands such as [Noskip](../Commands/Individual%20commands/Noskip.md). This is initialized to false in [life cycle > Dialogue setup](../life%20cycle.md#dialogue-setup).
