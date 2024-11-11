# LetterPrompt

Displays a prompt where a [flagstring](../../Flags%20arrays/flagstring.md) can be edited by changing each letter and redirects upon confirmation.

## Syntax

````
|letterprompt,flagstring,confirmline,promptline,maxlength|
````

## Parameters

### `flagstring`: int

The [flagstring](../../Flags%20arrays/flagstring.md) slot to edit. This must be a valid [flagstring](../../Flags%20arrays/flagstring.md) slot or an exception will be thrown.

### `confirmline`: int

This is the desired [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect when the letter prompt is confirmed. This must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

### `promptline`: int | `@`string

This is the desired string line or its corresponding [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to show as a prompt message. The int form must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown. The string form must begin with a `@` and not contain any SetText commands in it or the command parser will behave incorrectly and will likely cause an exception to be thrown.

### `maxlength`: int

The maximum amount of letters allowed to be entered. This must be a valid int or an exception will be thrown.

## Remarks

This command requires to be in [Dialogue mode](../Dialogue%20mode.md) or SetText will not wait for the prompt to finish and resume processing as normal which can cause undefined behaviors.

Before creating the letter prompt, this command will first assign [flagstring](../../Flags%20arrays/flagstring.md) 2 to this command's text, assign [flagvar](../../Flags%20arrays/flagvar.md) 5 to -1 and hide the current textbox with a shrink animation. Only then is the letter prompt created.

The handling of the letter prompt is done by MainManager's Update which restricts the inputs available to the 4 directions (to move the selected option), confirm, cancel (to erase the last character) and pause (to set the selected option to the confirm one). This is where the `maxlength` constraint is checked: if a letter is entered when the length of the current number text is the max, it will prevent the action. After the letter prompt is confirmed, the `flagstring` [flagstring](../../Flags%20arrays/flagstring.md) slot will have the value of the final result upon the confirmation.

Once the outcome is determined, control goes back to SetText inside the [Dialogue post-processing](../Life%20Cycle.md#dialogue-post-processing) where it will redirect the input string to `confirmline`.

### How letter prompt works at a high level

A letter prompt is a complex system that acts as a superset of [numberprompt](NumberPrompt.md), but with special fields set to differentiate it. There are multiple letter prompts with an id and a file corresponding to it that tells which letters can be entered. Here are the ids supported:

|Id|Description|
|--:|-----------|
|0|Latin alphabet with numbers, accented letters and common punctuation|
|1|Japanese's Katakana alphabet|
|2|Japanese's hiragana alphabet|
|3|Russian's cyrilic alphabet|
|4|Korean's Hangul alphabet split into 3 sections with its own logic|
|5|Some uncommon symbols such as music note and spade|

The current id can be cycled through using the Switch button and once done, it needs to recreate the whole prompt in a similar fashion than the first refresh which is the creation of the whole prompt. The Korean prompt is special as it has its own refreshing logic which is detailed in a section below.

The actual option index works very differently than typical prompts because while it is contiguous, the navigation involves rows and columns. The first row's length is considered to be the length of every row and it is calculated as the prompt is generated on refresh which allows to determine what option should be selected when using the directional inputs. A prompt's data file is located at `Assets/Ressources/Data` and has the name `LetterPromptX` where `X` is the id. The file contains all the letters the prompt offers where each row is delimited by LF. It may contain spaces, but those spaces won't be selectable: they are only used for visual separations. Since this wouldn't allow to enter a space, a special option is appended after the Erase one for inputting an actual space.

The important thing to note about a letter prompt is it is not possible to cancel it. It can only be confirmed and in that sense, it acts as a live [flagstring](../../Flags%20arrays/flagstring.md) editor because the backing [flagstring](../../Flags%20arrays/flagstring.md) changes as letters are removed or added. It's also the starting text which means it is both the input and the output.

Due to extreme uses of SetText, it is technically possible to run SetText commands if the `flagstring` contains any prior to the prompt appearing, but it is not possible to enter any because the `|` is not available in any prompt. By the same token, this prompt does not support putting commands inside `promptmessage` as it is not handled and it will break the parser.

One last thing to note: because the letter prompt consumes a lot of letter slots and nothing uses [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md), it should be noted that this UI in general is susceptible to do a lot of memory allocation and garbage collection which could be a performance concern.

### About the fields, [flagvar](../../Flags%20arrays/flagvar.md) and [flagstring](../../Flags%20arrays/flagstring.md) used

The way a letter prompt works is by reusing some fields used in the [prompt](Prompt.md) and [numberprompt](NumberPrompt.md) command as well as sharing some fields used by [ItemList](../../ItemList/ItemList.md). It's important to note that while it may shares some fields with these 2 systems, they are NOT equivalent semantically. This can in theory cause problems if they mix, but in practice, all fields mentioned here are properly reset by this command or their respective systems before usage so in practice, it's mostly safe.

Here are the fields in question:

* `letterprompt`: Set to to 1 in `Japanese`, 3 in `Russian`, 4 in `Korean` and 0 for any other [languageid](../languageid.md)
* `prompt`: Since this command uses the same lock then [prompt](Prompt.md), it works the same way here (set to true during the command processing and to false when SetText is done yielding and handles the outcome)
* `numberprompt`: Set to true during the command processing which is needed because a letter prompt is a superset of a [numberprompt](NumberPrompt.md) (it's not set to false because its value won't matter after until a [prompt](Prompt.md) command happens which itself would set it to false anyway)
* `option`: The current option index from 0 (includes spaces, but they are bypassed by the prompt handling)
* `maxoptions`: The amount of options total (NOTE: this isn't the same than [prompt](Prompt.md) or [ItemList](../../ItemList/ItemList.md) where it's the AMOUNT of options, not the index of the last one)
* `promptbox`: The 9box containing the prompt itself
* `npromptholder`: A GameObject that lives as a child of the `promptbox` and it contains the current text
* `listtype`: This is the field used to store the `flagstring` temporarily until the number prompt is confirmed. It does NOT correspond to an actual [listtype](../../ItemList/listtype.md).
* `listredirect`: This is the field used to store the `confirmline` until the number prompt is confirmed. It has nothing to do with its usage in [ItemList](../../ItemList/ItemList.md)
* `listcancel`: This field has the same value than `listredirect` as it is impossible to cancel a letter prompt. It has nothing to do with its usage in [ItemList](../../ItemList/ItemList.md).
* `listcancelled`: This field is set temporarily to true when erasing a letter and back to false when confirming any options.
* `multilist`: This has an unused purpose where if the letter prompt data had `}` in it, it would add the option to this list. It's set to a new list in practice still.

As for the [flagvar](../../Flags%20arrays/flagvar.md), this command and its handling uses the following slots:

* 0: Set to -555. This is to tell MainManager that this is a letter prompt, not a [numberprompt](NumberPrompt.md).
* 1: Set to the number of letters in each row of the current letter prompt
* 4: Set to 0 every time the letter prompt is refreshed (NOTE: why?)
* 5: Set to -1 from this command, then changes to the button id if it's a direction input during the prompt handling. This is used to skip any space selection by repeating the last action of the last direction pressed.
* 6: Set to the current section of the Korean prompt (id 4) if it's in use
* 10: Set to `maxlength`

For [flagstring](../../Flags%20arrays/flagstring.md):

* 0: Set to the current text which is necessary to track the length constraint by `maxlength`
* 1: Set to the letter prompt's data before generating the letter holder and after, it will be have the LF and { removed so each option index corresponds directly to the character in the string at that index.
* 2: Set to this command's text as a way to pass the parameters to CreateLetterPrompt on create and refresh.
* `flagstring`: Set to the current text after confirmation

### Letter prompt rendering

If the existing `promptbox` isn't null, it is first destroyed. This is because the same rendering procedure has to be done on refresh every time the letter prompt changes mode

The `promptbox` is a regular 9box rendered at (0.0, -1.25, 10.0) of size (15.0, 7.5) with a sortingOrder of -3 with the color of the textbox with a grow animation. Another 9box is childed to the `promptbox` with a position of (0.0, 5.0, 0.0), a size of (13.0, 2.0), and a sorting order of -3 without a grow animation. 

The `promptbox` contains the prompt message which has its SetText call done in non [Dialogue mode](../Dialogue%20mode.md) where the parent is the `promptbox` and the input string is |[center](Center.md)\| followed by the resolved text of `promptline` with a position of (0.0, 4.75, 9.0).

The `npromptholder` is a new GameObject named `letter holder` that lives as the child of the `promptbox`. Its local position to (0.0, 2.7, 0.05) without angles or scaling.

To render each options (the letters, Erase, Space and Confirm), the following is done:

* Start with the x and y position of the options be -6.35 and 1.8 respectively
* Render the options such that for each character in the letter prompt data (which is the value of [flagstring](../../Flags%20arrays/flagstring.md) 1 at this point):
    * If we encounter an LF:
        * Reset the x position to -6.35
        * Remove a certain amount from the y position. This amount is 0.75 if we aren't in the Korean letter prompt or 1.1 if we are and it's not the 4th line.
        * Set to no longer increment [flagvar](../../Flags%20arrays/flagvar.md) 1.
    * If we have yet to encounter the first LF:
        * Increment [flagvar](../../Flags%20arrays/flagvar.md) 1 (this means each row must have the same amount of options max than the first row as this value is going to be the number of options per row)
    * Have the option text be |[choicewave](Choicewave.md),n,true| where n is `maxoptions`.
    * Increment `maxoptions`.
    * Call [SetText](../SetText.md) in non [Dialogue mode](../Dialogue%20mode.md) using the text with |[center](Center.md)\|:
        * [fonttype](../Notable%20states.md#fonttype) is `BubblegumSans`
        * no `linebreak`
        * no `tridimensional`
        * `position` is (x, y, 0.0) where x and y are the current value of the option's positions.
        * no `camoffset`
        * `size` of (1.0, 1.0)
        * `parent` is the `promptbox`
        * no `caller`
    * Add 0.65 to x

After all the letters have been rendered, [flagstring](../../Flags%20arrays/flagstring.md) 1 will have its LF and { removed.

`maxoptions` is then incremented by 3 for the bottom 3 options. All the 3 options's SetText call are similar with the main differences between them is the position and the text, but they are all on [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode) with the `promptbox` as the parent with no size scaling or anything special. All are menutext prepanded with |[center](Center.md)\| and appended with |[choicewave](Choicewave.md),n| where n is the option index (they are the last 3 in order):

1. Cancel: menutext 74 at (-5.0, -3.0, 0.0)
2. Space: menutext 192 at (0.0, -3.0, 0.0)
3. Confirm: menutext 42 at (5.0, -3.0, 0.0). NOTE: On this one, if [languageid](../languageid.md) is 6 (`Russian`), the option text is prepanded with `|`[sizemulti](Sizemulti.md)`,0.8,1|`

After, the help text to the next letter prompt with the switch button is rendered by creating a new GameObject parented to the `promptbox` called `button` with a ButtonSprite set to `Scroll Faster / Switch Party` whose label is the corresponding letterPromptHelp at the current id at (-2.25, -1.9) and a size of (0.5, 0.5).

After the initial setup, the first refresh of the letter prompt is performed. This will destroy the text in `npromptholder` and rerender using the one in [flagstring](../../Flags%20arrays/flagstring.md) 0 which will get updated periodically as letters are added or removed during the prompt handling. This also sets [flagvar](../../Flags%20arrays/flagvar.md) 4 to 0. As for the rendering of the text itself, it is done via a SetText call in non [Dialogue mode](../Dialogue%20mode.md) with the text padded to the right with `_` to fit into `maxlength` prepended with |[center](Center.md)\|:

- [fonttype](../Notable%20states.md#fonttype) is `BubblegumSans`
- no linebreak
- no tridimensional
- position is Vector3.zero.
- no camoffset
- size of Vector2.one
- parent is the npromptholder
- no caller

After the first refresh is done, a refresh will only get done after a letter is added/removed during the letter prompt handling.

### About the Korean letter prompt (id 4)

The Korean prompt has special logic related to it. There are 3 sections of different parts of a full letter and one option of each section in order must be confirmed before the full letter is appended at the end of the current text. To accomplish this, there are special fields for this mainly [flagvar](../../Flags%20arrays/flagvar.md) 6 corresponds to the current section starting from 0, `KoreanHL` corresponds to the 2 parts selected in the first 2 sections and a hardcoded array of the mappings of each sections's options called `koreanLimit` (which starts at 3 instead of 0). This letter prompt also has its own refresh handling to render it correctly throughout the selection.

Just before the first prompt refresh mentioned above, the Korean one happens if we are using it. This will check each letter in the prompt:

* If the current selection includes the character, it is rendered with a pure red color
* If it corresponds to the current section, but no character was selected yet, it is rendered with a pure black color
* If it's a further section than the current one or the letter is in a past section, but isn't the selected one, it is rendered with a pure black color with half transparency

This refresh is then done every time the prompt refreshes if the prompt is the Korean one during the handling.

### Letter prompt handling

After the command has been processed, SetText will keep yielding frames in the [Dialogue post-processing](../Life%20Cycle.md#dialogue-post-processing) phase until `prompt` is set to false from MainManager's Update. This Update event completely changes the input handling of the game and in general, it restricts actions to interact with the current letter prompt. Additionally, the `blinker` will be disabled. 

Here are the dpad handlers:

* Up: Moves the cursor to option - [flagvar](../../Flags%20arrays/flagvar.md) 1 (one row above) clamped from 0 to `maxoptions` - 1 and set [flagvar](../../Flags%20arrays/flagvar.md) 5 to 0
* Down: Moves the cursor to option + [flagvar](../../Flags%20arrays/flagvar.md) 1 (one row below) clamped from 0 to `maxoptions` - 1 and set [flagvar](../../Flags%20arrays/flagvar.md) 5 to 1
* Left: Moves the cursor to the option - 1 (the one on the left) or to the one at the end of the current line (using [flagvar](../../Flags%20arrays/flagvar.md) 1 to find it) and set [flagvar](../../Flags%20arrays/flagvar.md) 5 to 2 (NOTE: this has an issue: if the current row doesn't have an option at the end which is the case for prompt id 1 and 4, it will select PAST the last option of the row which can end up in unexpected places or in an option index located past the last one)
* Right: Moves the cursor to the option + 1 (the one on the left) or to the one at the start of the current line (using [flagvar](../../Flags%20arrays/flagvar.md) 1 to find it) and set [flagvar](../../Flags%20arrays/flagvar.md) 5 to 3

After any dpad movement, if the option is still a letter, but it's a space, the option will change again as if the last direction will be pressed again (using [flagvar](../../Flags%20arrays/flagvar.md) 5 to determine it) until it isn't a space. The option is always clamped from 0 to `maxoptions` - 1 every time it is changed this way. [flagvar](../../Flags%20arrays/flagvar.md) 5 is then reset to -1.

For other inputs, they are restricted to the following:

* Pause: Move the cursor to the confirm option which is `maxoptions` - 1
* Switch: Rerender the letter prompt, but the id is now the current one + 1 or go back to 0 on the last one
* Cancel: Sets `listcancelled` to true. Then, if there are no letters entered, play the buzzer. Otherwise, the last character of the `flagstring` is removed and the letter prompt refreshed. This also resets the Korean prompt to its original state if we are in it
* Confirm: `listcancelled` is set to false, a 5 frames input cooldown is applied and it also set `promptpick` to 0. The rest depends on the option currently selected before resetting the [Backtracking](../Related%20Systems/Backtracking.md) system:
    * A letter: Add the entered letter to [flagstring](../../Flags%20arrays/flagstring.md) 0 (current text) and refresh the letter prompt unless the length is `maxlength` from [flagvar](../../Flags%20arrays/flagvar.md) 10 in which case, the buzzer plays. 
        * For the Korean prompt, the option must be within the current accepted section's range or the buzzer will play ([flagvar](../../Flags%20arrays/flagvar.md) 6 tells the current one, the accepted options ranges per sections are hardcoded). If it is in the valid range, then it depends on the current section:
          * Not last: [flagvar](../../Flags%20arrays/flagvar.md) 6 is incremented.
          * Last: [flagvar](../../Flags%20arrays/flagvar.md) 6 is reset to 0, the cumulation of all the letter selected in each sections is appended to [flagstring](../../Flags%20arrays/flagstring.md) 0, the current letter selections is cleared and the option index is reset to 0.
        * No matter what, if the selection was accepted, the prompt is refreshed and the Korean prompt logic is also refreshed
    * Erase: Remove the last entered letter from [flagstring](../../Flags%20arrays/flagstring.md) 0 (current text) and refresh the letter prompt unless the text was empty in which case, the buzzer plays.
        * For the Korean prompt, it also resets [flagvar](../../Flags%20arrays/flagvar.md) 6 (current section) to 0, clears the current selection and refresh the Korean prompt logic.
    * Space: Add a ` ` to [flagstring](../../Flags%20arrays/flagstring.md) 0 (current text) and refresh the letter prompt unless the length is `maxlength` from [flagvar](../../Flags%20arrays/flagvar.md) 10 or we are during a Korean prompt character selection ([flagvar](../../Flags%20arrays/flagvar.md) 6 is higher than 0) in which case, the buzzer plays.
    * Confirm: If [flagstring](../../Flags%20arrays/flagstring.md) 0 (current text) is empty, the buzzer is played, otherwise:
        * `promptpointers` is set to one element containing `confirmline` fetched from `listredirect`, [Text advance](../Related%20Systems/Text%20advance.md)'s `skiptext` is set to false and finally, the `prompt` lock is released

### Handling the letter prompt outcome

When SetText is done yielding, the prompt will be handled immediately after:

* The input string will be overwritten completely with the text obtained from the corresponding `promptpointers` associated with the option (which is always `confirmline` here since a letter prompt cannot be canceled) prepended with |[blank](Blank.md)\| and then [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) will be called on using the `linebreak` value if it isn't null.
* Set the current textbox to reveal itself with a grow animation.
* Setup the `promptbox` to have a shrink animation.
* Destroys the `promptbox` in 0.5 seconds.
* Reset `promptpick` to -1.
* Reset the character position of the char loop to restart at the beginning of the input string. This essentially restarts the whole char loop with the new input string within the same call then this command.
* Yield for a frame to let the animation plays.

After, processing continues as normal with a fresh input string once [Dialogue post-processing](../Life%20Cycle.md#dialogue-post-processing) is completed for this iteration of the char loop.
