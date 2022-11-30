This will describe at a high level the full processing steps of SetText when the (4) [Entry point](Entry%20point.md) executes.

## Setup

* Sets `bleepvolume` to `Size`'s magnitude
* Sets `tempoverf` to `overridefollower`
* Sets `camtoffset` to the MainManager's field value
* [textholder](Notable%20local%20variable/textholder.md) is initialized.
* Replace all `\r\n` by `\n` of the input string.
* If [fonttype](fonttype.md) is `BubblegumSans`, override it to `Uzura` if the [languageid](languageid.md) is `Japanese` and to `ONEMobilePOP` if [languageid](languageid.md) is `Korean`
* [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) the input string if `linebreak` has a value with.

## Dialogue setup

This phase only occurs if we are in [Dialogue mode](Dialogue%20mode.md) mode, otherwise,  a `|Spd,0|` is prepended to the input string which disables the wait times by default: [dialogue setup phase](Life%20Cycle/dialogue%20setup%20phase.md)

## Char loop

The char loop is the heart of SetText. It's a for loop on each char of the input string.

### Dialogue pre-processing

This phase of the loop only occurs in [Dialogue mode](Dialogue%20mode.md) mode:

* Sets MainManager.`fontdsize` to `Size`.x
* Sets MainManager.`fontdtype` to [fonttype](fonttype.md)
* Sets MainManager.linebr to `linebreak`'s value (no null check???)
* if we are processing one of the first 10 characters, the player isn't null and we are not in an event, sets the player's entity's flip to `initialflip` (why the first 10???)

### `\n` processing

This phase of the loop happens if the current char is a `\n`:

* Sets `currentoffset` to 0.0
* Sets `currentline` to 0.7f * `Size`.y * (([languageid](languageid.md) != `Japanese`) ? 1f : 1.25f)

### `|` proocessing

This phase of the loop happens if the current char is a `|` which is the delimiter for a [Commands](Commands/Commands.md):

* If in [Dialogue mode](Dialogue%20mode.md) mode, sets the [tailtarget](Notable%20local%20variable/tailtarget.md) to no longer be talking
* Sets `command` to be the string between the next char and the char before the next `|` which essentially reads the whole command with its parameters
* Sets `temp` to be `command` split by `,`. This means `temp[0]` is the command name itself while everything after is its parameters
* Sets `com` to the parsed [Commands](Commands/Commands.md) without case sensitivity using `temp[0]` where all the `.` have been removed
* If in [Dialogue mode](Dialogue%20mode.md) mode and `com` is among these [Commands](Commands/Commands.md): `Icon`, [Button](Commands/Individual%20commands/Button.md), `Size`, [Shaky](Commands/Individual%20commands/Shaky.md), [Wavy](Commands/Individual%20commands/Wavy.md), [Rainbow](Commands/Individual%20commands/Rainbow.md), [Glitchy](Commands/Individual%20commands/Glitchy.md), [Halfline](Commands/Individual%20commands/Halfline.md) or `Quarterline`, append the whole command call string with parameters and the `|`s to `tempdiag`
* Check to see if we are processing this command. The conditions are `Ignorenext` is 0 or below and we are not in [Testdiag](Commands/Individual%20commands/Testdiag.md)
  * If we are in [Testdiag](Commands/Individual%20commands/Testdiag.md), process the command only if it is allowed under dialogue test. If we are not in [Dialogue mode](Dialogue%20mode.md) mode, skips all command processing. For more information, check the [Testdiag](Commands/Individual%20commands/Testdiag.md) command documentation.
* **Process the [Commands](Commands/Commands.md) via a switch on `com` if we decided to process it**
* Decrement `Ignorenext` if it is higher than 0
* If we are resuming after the command is done, sets the current char to be the one after the command (after the second `|`). Otherwise, the command will continue at the same position (typically happens if the input string has changed).
* Resets the processing state after a command is done.

### ` ` processing

This phase of the loop happens if the current char is a ` `:

* Increment `written`
* Append a ` ` to `tempdiag`
* Add 0.3f * `Size`.x to `currentoffset`
* If we are in [Dialogue mode](Dialogue%20mode.md) mode, sets the [tailtarget](Notable%20local%20variable/tailtarget.md) to be talking
* if on [single letter rendering](Life%20Cycle/letter%20rendering/single%20letter%20rendering.md), append a ` ` to the single letter slot it uses.

### Letter processing

This phase of the loop occurs for everything else that isn't a `\n`, `|`, ` ` or `\r` (which are not processed in general):

* If in [Dialogue mode](Dialogue%20mode.md) mode and the [Speed](Commands/Individual%20commands/Speed.md) is 0.0, an `inputcooldown` of 5 frames is applied
* If `fontlock` is false, override [fonttype](fonttype.md) in [Dialogue mode](Dialogue%20mode.md) mode if `NumberPrompt` is false to `Uzura` if [languageid](languageid.md) is `Japanese` or the current character is Japanese or to `ONEMobilePOP` if [languageid](languageid.md) is `Korean` and the current character is Korean.
* Have a local var named fonttt which has the value of `fontttype`
* If not using [Single](Commands/Individual%20commands/Single.md) rendering
  * [regular letter rendering](Life%20Cycle/letter%20rendering/regular%20letter%20rendering.md)
* If we are using [Single](Commands/Individual%20commands/Single.md) rendering
  * [single letter rendering](Life%20Cycle/letter%20rendering/single%20letter%20rendering.md)
* If `Backbox` is not null
  * Sets the localPosition of the `Backbox` to `currentoffset` / 2.0, `Size`.y / 2.0 + 0.1, -5.0 and its localScale to `currentoffset` / 5.0 * `Size`.x, `Size`.y * 1.5, 1.0

### Dialogue post-processing

This phase of the loop only occurs in [Dialogue mode](Dialogue%20mode.md) mode:

* Sets the [tailtarget](Notable%20local%20variable/tailtarget.md) to no longer be talking
* As long as [Prompt](Commands/Individual%20commands/Prompt.md) is true and `ItemList` isn't null, yield a frame (which waits that the current prompt or ItemList goes inactive)
* if `prompttick` is above -1 and there is at least one `promptpointers` (which means a prompt needs to be handled)
  * [prompt handing](Life%20Cycle/special%20command%20handling/prompt%20handing.md)
* If `inlist` is true (which means a [Pickitem](Commands/Individual%20commands/Pickitem.md)'s ItemList needs to be handled or we are in an [inlist issue](../ItemList/inlist%20issue.md) situation)
  * [pickitem handling](Life%20Cycle/special%20command%20handling/pickitem%20handling.md)
* Sets the [tailtarget](Notable%20local%20variable/tailtarget.md) to be talking
* Sets `inlist` to false
* If the player isn't null, sets its npc to a new list

## Dialogue Cleanup

This phase only occurs if we are in [Dialogue mode](Dialogue%20mode.md) mode:
[dialogue cleanup phase](Life%20Cycle/dialogue%20cleanup%20phase.md)

## Cleanup

* If we are not in [Dialogue mode](Dialogue%20mode.md) and `Minibubble` is true and parent has a `Minibubble`, call DestroyThis on the `Minibubble` component
* Yield for a frame
* if [Transfer](Commands/Individual%20commands/Transfer.md) has a value, Call TransferMap with `transferti` as the map id
* if `eventcall` is above -1, call StartEvent with `eventtoss` as the event id and `caller` as the caller
* if `tokenbox` is not null, destroy it
* Remove a Game Token from key items
* if `caller` isn't null and it is an Object of type Item and its hit is true, Destroy it
* if `inlist` and [Prompt](Commands/Individual%20commands/Prompt.md) are false, set flags 349 to false
