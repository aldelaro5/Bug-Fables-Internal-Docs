# Next

Signals the game that the current textbox is over and wait for a confirmation input before resuming with a fresh textbox.

## Syntax

````
|next|
````

## Parameters

The parameters corresponds to the ones sent to a [Tail](Tail.md) command when the [Blank](Blank.md) command is processed at the end of processing this command. If no parameters are sent, the [Tail](Tail.md) command will not be processed at the end of the [Blank](Blank.md) command.

## Remarks

This commands will first reset `skiptext` and [Noskip](Noskip.md) to false (See [Text advance](../../Related%20Systems/Text%20advance.md)) and set the [tailtarget](../../Notable%20local%20variable/tailtarget.md) to stop talking. It will then yield control back to MainManager's Update by setting [waitinput](../../Global%20vars%20used/waitinput.md) to true and yielding until it becomes false. This causes the blinker to be enabled and for the standard input processing to occur until the textbox has been passed. When that happens, the text processed so far until this command is recorded for the [Backtracking](../../Related%20Systems/Backtracking.md) system if a backtracking wasn't in progress. The current backtracking if any is ended and finally, a [Blank](Blank.md) command is processed.

This command is essential to the [Backtracking](../../Related%20Systems/Backtracking.md) system as it is the only one that will give control back to the game allowing a backtracking to be initiated and it is also alongside [Goto](Goto.md) one of the 2 commands that delimits what a textbox text is.

This is one of the command supported by [Testdiag](Testdiag.md).
