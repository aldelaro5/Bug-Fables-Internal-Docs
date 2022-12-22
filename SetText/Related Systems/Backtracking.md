# Backtracking

[SetText](../SetText.md) supports a system called dialogue backtracking which allows in [Dialogue mode](../Dialogue%20mode.md) mode to see previous textbox that have been passed in the same SetText call.

This system is mainly possible using some important static field of MainManager:

* `currentdialogue`: Stores the index of the current textbox text being displayed (counted from 0).
* `diagstring`: Stores the textbox backtrack history. This includes the textbox SetText is waiting on if a backtracking is initiated.
* `tempdiag`: Accumulates a stripped version the current textbox text until it is done so it can be added to `diagstring`.
* `backtacking`: Tracks if a backtrack is in progress.

Everything starts by tracking the current textbox using `tempdiag` which serves as an accumulator.  During [dialogue setup phase](../Life%20Cycle/dialogue%20setup%20phase.md), it is set to |[size](../Commands/Individual%20commands/size.md),size.x,size.y| which serves as the starting value using the [size](../Commands/Individual%20commands/size.md) values. Then, it will accumulate every letters and spaces during processing, but only a subset of the commands will be accumulated. These commands are:

* [Icon](../Commands/Individual%20commands/Icon.md)
* [Button](../Commands/Individual%20commands/Button.md)
* [size](../Commands/Individual%20commands/size.md)
* [Shaky](../Commands/Individual%20commands/Shaky.md)
* [Wavy](../Commands/Individual%20commands/Wavy.md)
* [Rainbow](../Commands/Individual%20commands/Rainbow.md)
* [Glitchy](../Commands/Individual%20commands/Glitchy.md)
* [Halfline](../Commands/Individual%20commands/Halfline.md)
* [Quarterline](../Commands/Individual%20commands/Quarterline.md)
* [Line](../Commands/Individual%20commands/Line.md) (only in [regular letter rendering](../Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md))
* `Backline`

Any other command will not be accumulated to this static field.

The storage of the textbox in `diagstring` only happens upon a [Next](../Commands/Individual%20commands/Next.md) or [Goto](../Commands/Individual%20commands/Goto.md) command which delimits what constitutes a textbox. At the end of processing either command, `currentdialogue` is incremented and `tempdiag` added to `diagstring`. 

Additionally, `tempdiag` is reset to its starting value which is |[size](../Commands/Individual%20commands/size.md),size.x,size.y|. Every time [Next](../Commands/Individual%20commands/Next.md) is processed, but before adding the textbox to `diagstring`, it will yield control to MainManager.Update by setting [waitinput](../Global%20vars%20used/waitinput.md) to true. This is where the game allows a backtracking to be initiated, but only after at least one textbox has passed. It is also possible to backtrack on the last textbox if an [End](../Commands/Individual%20commands/End.md) command has not been processed.

Once a backtrack is initiated, the current line that SetText is waiting on [waitinput](../Global%20vars%20used/waitinput.md) to be released is added to `diagstring`, `currentdialogue` is decremented and `backtacking` is set to true. The reason adding the current is added is because since it has already been processed by SetText, [waitinput](../Global%20vars%20used/waitinput.md) can only come back to false once that current line is passed during a backtrack. The only way for this to happen is if the current line becomes a backtrack itself in `diagstring`.

From there, it is possible to freely go back and forth between the lines in `diagstring` which is tracked by `currentdialogue` that gets incremented and decremented accordingly. To render the text, all child of the [textbox](../Notable%20local%20variable/textbox.md) gets destroyed which blanks it and a separate SetText call is perform with the appropriate line from `diagstring`. This call is done with the string prepended with |[color](../Commands/Individual%20commands/Color.md),7| on the same [textbox](../Notable%20local%20variable/textbox.md) as the parent. This occurs in non dialogue mode which prevents conflicts with the fields mentioned above. This all happen every time a change in the textbox is performed.

Once the last textbox from `diagstring` is passed, MainManager.Update detects that `currentdialogue` and `diagstring`.Count are the same upon passing the textbox and releases the [waitinput](../Global%20vars%20used/waitinput.md). This causes the original SetText call to resume, but since `backtacking` hasn't been set to false yet, it knows to not add the current textbox since it has been added to `diagstring` already. At the very end of the [Next](../Commands/Individual%20commands/Next.md) processing, `backtacking` is finally set to false which allows SetText to continue processing as if nothing happened.
