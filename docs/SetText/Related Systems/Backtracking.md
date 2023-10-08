# Backtracking

[SetText](../SetText.md) supports a system called dialogue backtracking which allows in [Dialogue mode](../Dialogue%20mode.md) to see previous textbox that have been passed in the same SetText call. This system only works correctly in [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) because in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), it will not accumulate any letters to the history.

This system is mainly possible using some important static field of MainManager:

* `currentdialogue`: Stores the index of the current textbox text being displayed (counted from 0).
* `diagstring`: Stores the textbox backtrack history. This includes the textbox SetText is waiting on if a backtracking is initiated.
* `tempdiag`: Accumulates a stripped version of the current textbox text until it is done so it can be added to `diagstring`.
* `backtacking`: Tracks if a backtrack is in progress.

Everything starts by tracking the current textbox using `tempdiag` which serves as an accumulator.  During [Dialogue setup](../Life%20Cycle.md#dialogue-setup), it is set to |[size](../Individual%20commands/size.md),size.x,size.y| which serves as the starting value using the [size](../Individual%20commands/size.md) values. Then, it will accumulate every letters (except in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) where it's not supported) and spaces during processing, but only a subset of the commands will be accumulated. These commands are:

* [icon](../Individual%20commands/Icon.md)
* [button](../Individual%20commands/Button.md)
* [size](../Individual%20commands/size.md)
* [shaky](../Individual%20commands/Shaky.md)
* [wavy](../Individual%20commands/Wavy.md)
* [rainbow](../Individual%20commands/Rainbow.md)
* [glitchy](../Individual%20commands/Glitchy.md)
* [halfline](../Individual%20commands/Halfline.md)
* [quarterline](../Individual%20commands/Quarterline.md)
* [line](../Individual%20commands/Line.md) (only in [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md))
* [backline](../Individual%20commands/Backline.md)

Any other command will not be accumulated to this static field.

The storage of the textbox in `diagstring` only happens upon a [next](../Individual%20commands/Next.md) or [goto](../Individual%20commands/Goto.md) command which delimits what constitutes a textbox in backtracking. At the end of processing either command, `currentdialogue` is incremented and `tempdiag` added to `diagstring`. 

Additionally, `tempdiag` is reset to its starting value which is |[size](../Individual%20commands/size.md),size.x,size.y|. Every time [next](../Individual%20commands/Next.md) is processed, but before adding the textbox to `diagstring`, it will yield control to MainManager's Update by grabbing [waitinput](../Notable%20states.md#waitinput). This is where the game allows a backtracking to be initiated, but only after at least one textbox has passed. It is also possible to backtrack on the last textbox if an [end](../Individual%20commands/End.md) command has not been processed. Additionally, It is possible to disable any backtracking for happening by using the [Lockbacktrack](../Individual%20commands/Lockbacktrack.md) command.

Once a backtrack is initiated, the current line that SetText is waiting on [waitinput](../Notable%20states.md#waitinput) to be released is added to `diagstring`, `currentdialogue` is decremented and `backtacking` is set to true. The reason adding the current line is done is because since it has already been processed by SetText, waitinput can only come back to false once that current line is passed during a backtrack. The only way for this to happen is if the current line is added to backtracking itself in `diagstring`.

From there, it is possible to freely go back and forth between the lines in `diagstring` which is tracked by `currentdialogue` that gets incremented and decremented accordingly. To render the text, all child of the [textbox](../Notable%20states.md#textbox) gets destroyed which blanks it and a separate SetText call is performed with the appropriate line from `diagstring`. This call is done with the string prepended with |[color](../Individual%20commands/Color.md),7| on the same textbox as the parent. This occurs in [non dialogue mode](../Dialogue%20mode.md#non-dialogue-mode) which prevents conflicts with the fields mentioned above. This all happen every time a change in the textbox is performed.

Once the last textbox from `diagstring` is passed, MainManager's Update detects that `currentdialogue` and `diagstring.Count` are the same upon passing the textbox and releases the waitinput. This causes the original SetText call to resume, but since `backtacking` hasn't been set to false yet, it knows to not add the current textbox since it has been added to `diagstring` already. At the very end of the [next](../Individual%20commands/Next.md) processing, `backtacking` is finally set to false which allows SetText to continue processing as if nothing happened.
