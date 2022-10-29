# Prompt

Display a prompt offering multiple options with cancellation support, then wait that an option has been chosen and process it in the same SetText call.

## Syntax

(1)

````
|prompt,yesno,yespoointer,nopointer|
````

(2)

````
|prompt,yesno,yespoointer,nopointer,canceloption|
````

(3)

````
|prompt,type,xminsizeyoffset,maxoptions,promptpointers...,prompttexts...,canceloption|
````

## Parameters

### `type`: `yesno` | `map` | `card` | `main` | `menu`

The type of the prompt to use which dictates what `promptpointers` and `prompttexts` (in int form) refers to (this is case sensitive):

* `yesno`: An alias for (3) where the `type` is `map`, `xminsizeypos` is 0.5 (which is a dummy value), `maxoptions` is 2, `promptpointers` are the `yespointer` and `nopointer` values respectively, `prompttexts` are the yes and no line ids from MenuText respectively and `canceloption` will be the one provided in (2) or default to 1 in (1) which is the no option.
* `map`: The `promptpointers` and the int form of `prompttexts` refers to [Dialogue line id](../Dialogue%20line%20id.md)s. If any pointer is an invalid id, an exception will be thrown.
* `card`: The `promptpointers` refers to a CardDialogue line id and the int form of `prompttexts` refers to a [Dialogue line id](../Dialogue%20line%20id.md)s. If any pointer is an invalid id or a cardgame isn't in progress, an exception will be thrown.
* `main`, `menu`: The `promptpointers` and the int form of `prompttexts` refers to a MenuText line id. If any pointer is an invalid id, an exception will be thrown.

Any other value will default to `map`.

### `yespointer`: int

The [Dialogue line id](../Dialogue%20line%20id.md) for the yes option in (1) or (2). This parameter is not applicable to (3). This value must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

### `nopointer`: int

The [Dialogue line id](../Dialogue%20line%20id.md) for the no option in (1) or (2). This parameter is not applicable to (3). This value must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

### `xminsizeyoffset`: array split by `;`: `x`float | `$`float

This allows to specify a minimum horizontal size and a vertical offset when rendering the prompt. The last `x`float element refers to the the former while the last `$`float element refers to the later. Any other element prefixes will cause the element to be ignored and the float part of the elements must be valid float values or an exception will be thrown. Any separator other than `;` will be ignored and be parsed as if it was part of the element. Either values are optional and it is possible to provide a dummy value to this parameter to not specify either. The order of the values in the array does not matter, but if a value is specified in multiple element, the last one takes priority.

### `maxoptions`: int

The number of options this prompt has. This value must be a valid int value or an exception will be thrown. Additionally, this value dictates the amount of `promptpointers` and `prompttexts` the command has. If this value is higher than the amount of either of these parameters, an exception will be thrown and if it is lower than either, SetText will only read the amount specified by this parameter and ignore the remaining ones.

### `promptpointers`: int...

The line ids of each of the prompt options. The amount to specify must be equal or higher than `maxoptions` and each must be a valid line id value or an exception will be thrown. The type of line id to use is dictated by the `type` value.

### `prompttexts`: `@`string... | int

The displayed texts or line id of each of the prompt options. The amount to specify must be equal or higher than `maxoptions` and each int form must be a valid line id value or an exception will be thrown. For the `@`string form, this supplies the text directly with inline commands support. Every `}` will be replaced by `,` and every `{` will be replaced by `|` which allows the command parser to process commands from a separate SetText after parsing this prompt one. This form takes priority over the int one and any other prefix of the string will be ignored and the value will be taken as an int.

### `canceloption`: `none` | `-1` | `$`int

The cancel option index for the `$`int form or disables the ability to cancel with `none` or `-1`. Specifying `$-1` is equivalent to `-1` or `none`. Otherwise, the int must be between 0 and `maxoptions` - 1. Any other int will cause an exception to be thrown upon the prompt's option being chosen. Any other value or any int prefixed with anything other than `$` will be ignored and the value will default to the equivalent of `none` or `-1`. If this option is not specified, the last parameter sent to this command (which is the last `prompttexts` value) is used as this parameter's value instead.

## Remarks

This command requires to be in [Dialogue mode](../../Dialogue%20mode.md) mode or SetText will not wait for the prompt to finish and resume processing as normal which can cause undefined behaviours.

For the `xminsize` part of `xminsizeyoffset`, the horizontal length of the prompt is calculated by taking the longest among the `prompttexts` text. If that length is smaller than the `xminsize`, the `xminsize` will be the horizontal length of the prompt. Otherwise, the calculated length takes precedence.

### Prompt handling

While this command manages the creation and setup of the prompt, its handling is left to be handled in MainManager's Update. There are several MainManager instance fields in this command to make this possible:

* `promptbox`: The 9box containing the prompt, created during processing using `xminsizeyoffset` and the texts obtained from `prompttexts` using separate SetText calls for each. These calls are done with the following parameters:
  * The input string is the text value appended with `|choicewave,o|` where o is the corresponding option index. 
  * [fonttype](../../fonttype.md) is `BubblegumSans`
  * `linebreak` is null
  * [Dialogue mode](../../Dialogue%20mode.md) is false (this is to prevent SetText from waiting on the prompt as `prompt` will be true when this call is made)
  * `tridimensional` is false
  * `position` is (0.0 - `xminsize` - 0.1, `yoffset`, 0.0)
  * `cameraoffset` is Vector3.Zero
  * `Size` is (0.85, 0.85, 1.0)
  * [Parent](Parent.md) is the `promptbox` itself
  * `caller` is null
* `prompt`: The flag tracking if a prompt is active, set to true during this command's processing and set to false by MainManager's Update once an option has been chosen.
* `promptpick`: The option index chosen, already been set to -1 from the [life cycle > Setup](../../life%20cycle.md#setup) phase and set by MainManager's Update when an option has been chosen to `option` (if a cancel was performed, `listcancel` will be used instead).
* `promptpointers`: The pointers specified by `promptpointers` which are set during the processing of this command.
* `maxoptions`: The amount of option the prompt has which is set from this command's `maxoptions` during processing.
* `option`: The option index the cursor is currently at, initialized to 0 from this command's processing.
* `listcancel`: The option index that will be the promptpick if a cancel is performed or -1 if cancelling is disabled, set by `canceloption` during this command's processing (-1 is set if the value is `-1` or `none`).

Just before ending the command processing, a 5 frame input cooldown is applied which allows the prompt to appear on the screen. After the command has been processed, SetText will keep yielding frames in the [life cycle > Dialogue post-processing](../../life%20cycle.md#dialogue-post-processing) phase until `prompt` is set to false from MainManager's Update. This Update event completely changes the input handling of the game and in general, it restricts actions to interact with the current prompt. Additionally, the `blinker` will be disabled. The actions will be restricted to the following:

* Up: Moves the cursor to the option above or to the last option if it was on the first one (also sets `option` accordingly).
* Down: Moves the cursor to the option below or to the first option if it was on the last one (also sets `option` accordingly).
* Cancel: If `listcancel` is not -1, sets `prompt` to false with `listcancel` as the `promptpick` and play the cancel sound, otherwise, nothing happens.
* Confirm: Sets `prompt` to false and `promptpick` to `option` and play the confirm sound.

Once the prompt handling is complete, MainManager.Update will set `skiptext` to false, apply an input cooldown of 10 frames on cancel or 5 frames on confirm. revert its input processing to the standard method when [message](../../Global%20vars%20used/message.md) is true.

### Handling the chosen option

When SetText is done yielding, the prompt will be handled immediately after:

* If flagvar 0 is set to -555, [textbox](../../Notable%20local%20variable/textbox.md) will be forced to not be shrunk. This is expected to be set before this command is processed.
* The input string will be overwritten completely with the text obtained from the corresponding `promptpointers` associated with the option prepended with `|blank|` and then [OrganiseLines](../../Notable%20Methods/OrganiseLines.md) will be called on it if `linebreak` is not null.
* Setup the `promptbox` to have a shrink animation.
* Destroys the `promptbox` in 0.5 seconds.
* Reset `promptpick` to -1.
* Reset the character position of the char loop to restart at the beginning of the input string. This essentially restarts the whole char loop with the new input string within the same call then this command.
* Yield for a frame to let the animation plays.

After, processing continues as normal with a fresh input string once [life cycle > Dialogue post-processing](../../life%20cycle.md#dialogue-post-processing) is completed for this iteration of the char loop.
