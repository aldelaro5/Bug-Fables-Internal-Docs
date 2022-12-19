# NumberPrompt

Displays a prompt where an integer number can be entered by entering each digits and redirects upon confirmation or cancellation. The result is stored in a [flagvar](../../../Flags%20arrays/flagvar.md) slot upon confirmation.

## Syntax

````
|numberprompt,outflagvar,naxlength,confirmline,cancelline|
````

## Parameters

### `outflagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot to place the result upon confirmation. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `maxlength`: int

The maximum amount of digits allowed to be entered. Assigned to [flagvar](../../../Flags%20arrays/flagvar.md) 10. This must be a valid int or an exception will be thrown.

### `confirmline`: int

This is the desired [Dialogue line id](../Dialogue%20line%20id.md) to redirect when the number prompt is confirmed. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

### `cancelline`: int

This is the desired [Dialogue line id](../Dialogue%20line%20id.md) to redirect when the number prompt is cancelled. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

This command requires to be in [Dialogue mode](../../Dialogue%20mode.md) mode or SetText will not wait for the prompt to finish and resume processing as normal which can cause undefined behaviors.

This command will first setup a number prompt to be displayed and yield until its outcome is determined using the same prompt lock used by [Prompt](Prompt.md). 

The handling of the number prompt is done by MainManager's Update which restricts the inputs available to the 4 directions (to move the selected), confirm, cancel and pause (to set the selected option to the confirm one). This is where the `maxlength` constraint is checked: if a number is entered when the length of the current number text is the max, it will prevent the action. If the number prompt is confirmed, the `outflagvar` [flagvar](../../../Flags%20arrays/flagvar.md) slot is assigned to the result before releasing the prompt lock. 

Once the outcome is determined, control goes back to SetText inside the [life cycle > Dialogue post-processing](../../life%20cycle.md#dialogue-post-processing) where it will redirect the input string to `cancelline` if cancelled and to `confirmline` if confirmed.

### About the fields, [flagvar](../../../Flags%20arrays/flagvar.md) and [flagstring](../../../Flags%20arrays/flagstring.md) used

The way a number prompt works is by reusing some fields used in the [Prompt](Prompt.md) and `LetterPrompt` command as well as sharing some fields used by [ItemList](../../../ItemList/ItemList.md). It's important to note that while it may shares some fields with these 2 systems, they are NOT equivalent semantically. This can in theory cause problems if they mix, but in practice, all fields mentioned here are properly reset by this command or their respective systems before usage so in practice, it's mostly safe.

Here are the fields in question:

* `letterprompt`: Set to -1 to indicate this is not a `letterprompt` which is a superset of number prompt
* `listcancelled`: Set to false during the command processing and to true if the number prompt was cancelled
* `prompt`: Since this command uses the same lock then [Prompt](Prompt.md), it works the same way here (set to true during the command processing and to false when SetText is done yielding and handles the outcome)
* `numberprompt`: Set to true during the command processing (it's not set to false because its value won't matter after until a [Prompt](Prompt.md) command happens which itself would set it to false anyway)
* `option`: The current option index from 0 to 11
* `maxoptions`: The index of the last option being 11 here (NOTE: this isn't the same than [Prompt](Prompt.md) or [ItemList](../../../ItemList/ItemList.md) where it's the AMOUNT of options)
* `promptbox`: The 9box containing the prompt itself
* `npromptholder`: A GameObject that lives as a child of the `promptbox` and it contains the current number text
* `listtype`: This is the field used to store the `outflagvar` temporarily until the number prompt is confirmed. It does NOT correspond to an actual [listtype](../../../ItemList/listtype.md).
* `listredirect`: This is the field used to store the `confirmline` until the number prompt is confirmed. It has nothing to do with its usage in [ItemList](../../../ItemList/ItemList.md)
* `listcancel`: This is the field used to store the `cancelline` until the number prompt is cancelled. It has nothing to do with its usage in [ItemList](../../../ItemList/ItemList.md)

As for the [flagvar](../../../Flags%20arrays/flagvar.md), this command and its handling uses the following slots:

* `outflagvar`: Will be used to output the result only if the prompt is confirmed (it is not touched if the prompt is cancelled)
* 0: Will be set to 0 if its value was -555 before. This is to prevent this prompt from being seen as a `LetterPrompt` which is a superset of number prompt.
* 4: Set to 0 every time the number prompt is refreshed (NOTE: why?)
* 5: Set to -1 from this command, then changes to the button id if it's a direction input during the prompt handling. At the end of the prompt, this value will correspond to the last direction pushed during the prompt handling NOTE: why?
* 10: Set to `maxlength`

This command also uses [flagstring](../../../Flags%20arrays/flagstring.md) 0 to store the number's text while the number prompt is being handled. Its integer representation is assigned to the `outflagvar` [flagvar](../../../Flags%20arrays/flagvar.md) slot when the number prompt is confirmed. This temporary storage is necessary to track the length constraint by `maxlength`.

### Number prompt rendering

The `promptbox` is a regular 9box rendered at (0.0, -1.2, 10.0) of size (7.0, 3.0) with a sortingOrder of -3 with the color of the textbox with a grow animation

The `npromptholder` is a new GameObject named `number holder` that lives as the child of the `promptbox`. Its local position to (0.0, 0.5, 0.05) without angles or scaling.

To render each 12 options (the 10 digits, Confirm and Erase), the following is done:

* Start with the x and y position of the options be -2.25 and -0.25 respectively
* Render the options such that for each options:
  * Have each text be |[Choicewave](Choicewave.md),n,true| where n is the option index from 0 to 11. This is prepanded with menutext 42 (`Confirm`) for option 10 and menutext 74 (`Erase`) for option 11. Option 0-9 are the digits.
  * Call [SetText](../../SetText.md) in non [Dialogue mode](../../Dialogue%20mode.md) using the text prepanded with |[Center](Center.md)\|:
    * [fonttype](../../fonttype.md) is `BubblegumSans`
    * no linebreak
    * no `tridimensional`
    * position is (x, y, 0.0) where x and y are the current value of the option's positions.
    * no camoffset
    * size of (0.85, 0.85)
    * parent is the promptbox
    * no caller
  * Add 0.5 to x
  * if we had just rendered option 9
    * Remove -1.5 to x and 0.75 to y
  * If we had just rendered option 10
    * Set x to 1.5

After the initial setup, the first refresh of the number prompt is perform. This will destroy the text in `npromptholder` and rerender using the one in [flagstring](../../../Flags%20arrays/flagstring.md) 0 which will get updated periodically as digits are added in removed during the prompt handling. This also sets [flagvar](../../../Flags%20arrays/flagvar.md) 4 to 0. As for the rendering of the number text itself, it is done via a SetText call in non [Dialogue mode](../../Dialogue%20mode.md) with the text padded to the right with `_` to fit into `maxlength` prepended with |[Center](Center.md)\|:
- [fonttype](../../fonttype.md) is `BubblegumSans`
- no linebreak
- no `tridimensional`
- position is Vector3.zero.
- no camoffset
- size of Vector2.one
- parent is the npromptholder
- no caller

After the first refresh is done, a refresh will only get done after a digit is added/removed during the number prompt handling.

### Number prompt handling

Just before ending the command processing, a 5 frame input cooldown is applied which allows the number prompt to appear on the screen. After the command has been processed, SetText will keep yielding frames in the [life cycle > Dialogue post-processing](../../life%20cycle.md#dialogue-post-processing) phase until `prompt` is set to false from MainManager's Update. This Update event completely changes the input handling of the game and in general, it restricts actions to interact with the current number prompt. Additionally, the `blinker` will be disabled. 

The actions will be restricted to the following:

* Up: Moves the cursor to option 0 (digit 0) and set [flagvar](../../../Flags%20arrays/flagvar.md) 5 to 0
* Down: Moves the cursor to option 11 (Confirm) and set [flagvar](../../../Flags%20arrays/flagvar.md) 5 to 1
* Left: Moves the cursor to the option below the current one and set [flagvar](../../../Flags%20arrays/flagvar.md) 5 to 2
* Right: Moves the cursor to the option above the current one and set [flagvar](../../../Flags%20arrays/flagvar.md) 5 to 3
* Pause: Move the cursor to the confirm option
* Cancel: Sets `listcancelled` to true, `promptpick` to 0, `promptpointers` to be one element containing `cancelline` fetched from `listcancel` and [Text advance](../../Related%20Systems/Text%20advance.md)'s `skiptext` to false. This will also apply a 15 frames inputcooldown, reset the [Backtracking](../../Related%20Systems/Backtracking.md) system and finally, release the `prompt` lock
* Confirm: A 5 frames input cooldown is applied and it also set `promptpick` to 0. The rest depends on the option currently selected before resetting the [Backtracking](../../Related%20Systems/Backtracking.md) system:
  * A number: Add the entered digit to [flagstring](../../../Flags%20arrays/flagstring.md) 0 (current number text) and refresh the number prompt unless the length is `maxlength` from [flagvar](../../../Flags%20arrays/flagvar.md) 10 in which case, the buzzer plays
  * Erase: Remove the last entered digit from [flagstring](../../../Flags%20arrays/flagstring.md) 0 (current number text) and refresh the number prompt unless the text was empty in which case, the buzzer plays
  * Confirm: If [flagstring](../../../Flags%20arrays/flagstring.md) 0 (current number text) is empty
    * This is treated the same way than if cancel was pressed with the exception that the input cooldown is 5 frames instead of 15 and `listcancelled` isn't set to false. 
    * If it wasn't empty, the `outflagvar` [flagvar](../../../Flags%20arrays/flagvar.md) slot is set to the integer version of [flagstring](../../../Flags%20arrays/flagstring.md) 0 (the current number text), `promptpointers` is set to one element containing `confirmline` fetched from `listredirect`, [Text advance](../../Related%20Systems/Text%20advance.md)'s `skiptext` is set to false, the [Backtracking](../../Related%20Systems/Backtracking.md) system is reset and finally, the `prompt` lock is released

### Handling the number prompt outcome

When SetText is done yielding, the prompt will be handled immediately after:

* The input string will be overwritten completely with the text obtained from the corresponding `promptpointers` associated with the option prepended with |[blank](Blank.md)\| and then [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) will be called on using the linebreak value if it isn't null.
* Setup the `promptbox` to have a shrink animation.
* Destroys the `promptbox` in 0.5 seconds.
* Reset `promptpick` to -1.
* Reset the character position of the char loop to restart at the beginning of the input string. This essentially restarts the whole char loop with the new input string within the same call then this command.
* Yield for a frame to let the animation plays.

After, processing continues as normal with a fresh input string once [life cycle > Dialogue post-processing](../../life%20cycle.md#dialogue-post-processing) is completed for this iteration of the char loop.