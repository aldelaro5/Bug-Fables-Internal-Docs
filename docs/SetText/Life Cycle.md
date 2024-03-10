# SetText lifecycle
This will describe at a high level the full processing steps of [SetText](SetText.md) when the last [entry point](Entry%20Points.md) executes.

## Setup

This phase happens at the start of every call. It mainly initialises [Commands](Commands.md) specific temporary values and saves some fields that can change during the course of this call for restore later in the cleanup phase. It also perform universal initialization logic such as setting up defaults values.

Here are the values saved for restoring them in cleanup:

* MainManager's `camoffset` This can be changed using the cameraoffset parameter in [Dialogue mode](Dialogue%20mode.md) or the [camoffset](Individual%20commands/Camoffset.md) command in any call, but the latter is only restored in dialogue cleanup in [Dialogue mode](Dialogue%20mode.md).
* MainManager's `overridefollower`. This can changed using the [overfollower](Individual%20commands/Overfollower.md) command, but it is only restored in dialogue cleanup in [Dialogue mode](Dialogue%20mode.md) and the restore can be forced to true by using the [Event](Individual%20commands/Event.md) command in syntax (1).

The call starts with the current bleep sound volume set to the size's parameter's magnitude. 

The [fonttype](Notable%20states.md#fonttype) parameter is overridden in some languages if `BubblegumSans` was sent:

* `Japanese`: `Uzura`
* `Korean`: `ONEMobilePOP`

Then, the [textholder](Notable%20states.md#textholder.md) is initialized where its parent and position is set to their respective parameters. Finally, all line endings of the input string are normalized to LF if any were in CRLF and [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) is called if the linebreak parameter wasn't null.

## Dialogue setup

This phase only occurs if we are in [Dialogue mode](Dialogue%20mode.md), otherwise,  a |[spd](Individual%20commands/Spd.md),0| is prepended to the input string which disables the wait times by default

This phase is where all the initialization logic needed for [Dialogue mode](Dialogue%20mode.md) are done as well as some defaults values logic. The nature of this phase is why only a single [Dialogue mode](Dialogue%20mode.md) call can be running at a given time with the narrow exception of the [battle](Individual%20commands/Battle.md) and [cardbattle](Individual%20commands/Cardbattle.md) commands during the yield. 

To enforce this and to setup some logic needed during the dialogue, this phase grabs/releases locks as well as interacts with entities:

* [message](Notable%20states.md#message) lock is grabbed which is the main one that enforces one SetText call in [Dialogue mode](Dialogue%20mode.md) at a time. 
* `minipause` lock is also grabbed,
* [waitinput](Notable%20states.md#waitinput) lock is released here which resets it from a previous call.
* `StopMovingEntities` is called on the map if it isn't null. This would pause most activities on the map while the dialogue is ongoing.
* The whole party and their followers's forcemove are stopped with the exception of the player and every party's entity's sprite is enabled except the lead.
* Make the player stop moving if it isn't null.

This phase also sets some values as defaults:

* |[size](Individual%20commands/size.md),0.8,0.9| gets prepended to the input string if [languageid](languageid.md) is `Japanese` (this doesn't affect [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) because it is disabled in that language)
* [speed](Individual%20commands/Speed.md) / [spd](Individual%20commands/Spd.md): 0.02 or 0.03 in `Japanese`
* The current [Prompt](Individual%20commands/Prompt.md), [NumberPrompt](Individual%20commands/NumberPrompt.md) and [LetterPrompt](Individual%20commands/LetterPrompt.md)'s `promptpick` is reset to -1 which allows a fresh prompt to be used.
* The cameraoffset's parameter is added to the current `camoffset` if its magnitude is large enough (0.1).
* [tailtarget](Notable%20states.md#tailtarget): Set to the parent's parameter.
* [bleep](Individual%20commands/Bleep.md): The pitch and type are set from the parent's paremeter if it exists.
* [textbox](Notable%20states.md#textbox): It is initialized here with the default [boxstyle](Individual%20commands/Boxstyle.md) with the ability to be hidden/shrunk and revealed/grown using `DialogueAnim`:
    * Parent: the GUICamera
    * No angle and scale
    * Position at (0.0, 3.25, 10.0)
* [textholder](Notable%20states.md#textholder): Has different defaults than setup:
    * Parent: [textbox](Notable%20states.md#textbox)
    * No angles and scale
    * Position at (-5.5, 0.9, 0.0) or the position parameter if its magnitude is large enough (0.1)
* The MainManager's version of [textbox](Notable%20states.md#textbox): The [textholder](Notable%20states.md#textholder) which allows global access.
* `blinker`: Initialized here.

Additionally, this phase includes some logic about the HUD: all of them are hidden except the berry count only if the caller's parameter isn't null, its `interacttype` is `Shop` or `CaravanBadge` and its `dialogues[1].y` isn't 1, but it is hidden otherwise

Some special logic also happens if the player isn't null and we're not in an event:

* Make the player face the caller (or the caller's shopkeeper if its  `interacttype` is `Shop` or `CaravanBadge`)
* Set `initialflip` to the player's flip.
* Saves the player's rigid body's constraints to a temporary variable.
* if the caller's parameter isn't null, teleport following players and chompy closer to caller? (2.55 max distance, 1.5 away?). After, yield for a frame.

Finally, this phase also does the following:

* Apply an `inputcooldown` of 10 frames.
* Setup the [GlobalCommand](Related%20Systems/GlobalCommand.md) system if the current map isn't null and supports it.

Before going to the char loop, this will yield until the `switchcooldown` expires if there is one.

## Char loop

The char loop is the heart of SetText. It's a for loop on each char of the input string. It is not a linear loop however: [Commands](Commands.md) have the ability to not only reprocess the same index when the command's text is replaced and resume from there, but it is also possible to replace the entire input string and resume processing at the start of it. That action action is known as a redirect. It is up to the command to decide what to signal the char loop which will be taken into account on the next iteration.

### Dialogue preprocessing

This phase of the char loop only occurs in [Dialogue mode](Dialogue%20mode.md). This is where the current horizontal size, the [fonttype](Notable%20states.md#fonttype) and the line break values are saved into MainManager's fields. This is needed by the [backtracking](Related%20Systems/Backtracking.md) system to add the [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the current accumulator if we initiate a backtracking because OrganiseLines needs the values at the start of the char loop, not the ones once a [next](Individual%20commands/Next.md) is yielding because they could have changed.

if we are processing one of the first 10 characters, the player isn't null and we are not in an event, sets the player's entity's `flip` to its initial value that was set during the dialogue setup or the last [align](Individual%20commands/Align.md) command (TODO: why the first 10?)

### LF processing

This phase of the loop happens if the current char is a `\n`. 

The current offset will be reset to 0.0 (the start of the line) and the current line to 0.7 * the vertical size's value (On `Japanese`, this is also multiplied by 1.25).

### Vertical bar processing

This phase of the loop happens if the current char is a `|` which is the delimiter for a [Commands](Commands.md). The command is first parsed into 2 parts: its enum value (case insensitive) and its syntax string (which includes both `|`, the command's name and all its parameters split by `,`). The command text is then accumulated to the [backtracking](Related%20Systems/Backtracking.md) system if the command supports it.

Not all commands will be processed. Notably, if [ignorenext](Individual%20commands/Ignorenext.md) is active, the command will be ignored. Same goes for [testdiag](Individual%20commands/Testdiag.md) which restricts commands processing to only work in [Dialogue mode](Dialogue%20mode.md) and to commands that it supports. For the actual command's processing, check the [Commands](Commands.md) documentation to learn more about each of them.

After the command has been processed (or ignored), this is where the next position of the char loop is managed. Normally, it increments, but if the command changes it or wish to resume at the same position, it will not be incremented.

### Space processing

This phase of the loop happens if the current char is a ` `. The space is accumulated in the [backtracking](Related%20Systems/Backtracking.md) system as well as in the current letter slot if we are using [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md). The current offset is incremented by 0.3 * the horizontal size's value which will essentially the next letter with a physical gap in [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md). Finally, `written` is incremented ??? and the [tailtarget](Notable%20states.md#tailtarget) is set to be talking if we were in [Dialogue mode](Dialogue%20mode.md).

### Letter processing

This phase of the loop occurs for everything else that isn't a `\n`, `|`, ` ` or `\r` (the last one isn't processed in general). This is where the letter gets rendered as well as some additional logic.

The first thing it does is if we had [speed](Individual%20commands/Speed.md) set to 0 (instant text scrolling) in [Dialogue mode](Dialogue%20mode.md), am `inputcooldown` of 5 frames is applied. This enforces a wait of at least 5 frames before the next [waitinput](Notable%20states.md#waitinput) can be passed even if the scrolling speed is instant.

Next, the [fonttype](Notable%20states.md#fonttype) is overridden again when in [Dialogue mode](Dialogue%20mode.md) and not in a [NumberPrompt](Individual%20commands/NumberPrompt.md) using a similar logic than the setup phase and the [languageid](languageid.md), but this time, this will take into consideration the font locking feature of the [font](Individual%20commands/Font.md) command. If the lock was enabled, this will not do the override:

* `Japanese` and the letter being processed character is Japanese: `Uzura`
* `Korean` and the letter being processed character is Korean: `ONEMobilePOP`

From there, the rendering of the letter occurs. There are 2 rendering methods and it depends on the current state of [single](Individual%20commands/Single.md). Each method is documented in its own page:

* If not using single rendering
    * Render the letter using [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md)
* If we are using single rendering
    * Render the letter using [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)

After the render is done, if a [backbox](Individual%20commands/Backbox.md) was processed, the position of the backbox is set to (current offset / 2.0, size.y / 2.0 + 0.1, -5.0) and its scale to (current offset / 5.0 * size.x, size.y * 1.5, 1.0)

### Dialogue post-processing

This phase of the loop only occurs in [Dialogue mode](Dialogue%20mode.md). It mainly includes special handling logic which mostly takes effect after specific commands as well as interact with with the [tailtarget](Notable%20states.md#tailtarget):

* Sets the tailtarget to no longer be talking
* As long as we are in a [Prompt](Individual%20commands/Prompt.md), [NumberPrompt](Individual%20commands/NumberPrompt.md), [LetterPrompt](Individual%20commands/LetterPrompt.md) or [ItemList](../ItemList/ItemList.md) yield a frame (this waits it goes inactive)

From there, there are 2 special handlers that run when applicalble: prompt handling and [Pickitem](Individual%20commands/Pickitem.md) handling, but no matter which one is ran, the following happens after:

* Sets the [tailtarget](Notable%20states.md#tailtarget) to be talking again.
* Reset [ItemList](../ItemList/ItemList.md)'s `inlist` to false.
* If the player isn't null, sets its NPC to a new list

#### Prompt handling

The prompt handler is run if the `promptpick` is higher than -1 with more than one `promptpointers` which only happens after the prompt, whatever its type, just went inactive, but still needs to be handled:

* if it was a [LetterPrompt](Individual%20commands/LetterPrompt.md) (using[flagvar](../Flags%20arrays/flagvar.md) 0 being -555 as a check), set the [textbox](Notable%20states.md#textbox) to unhide itself.
* Perform a redirection where the input string is the selected prompt's choice's text prepended with |[blank](Individual%20commands/Blank.md)\|.
* [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) the input string appended with the [questprompt](Individual%20commands/Questprompt.md) string if applicable.
* Sets the `promptbox` to hide itself and be destroyed in 0.5 seconds
* Reset the `promptpick` to -1
* Signal SetText to resume processing at the start of the new input string.
* yield a frame

#### [ItemList](../ItemList/ItemList.md) handling

For the ItemList handler, it only runs if `inlist` is true and `listredirect` isn't null or -1 which normally happens when the ItemList of a [pickitem](Individual%20commands/Pickitem.md) command goes inactive, but still needs to be handled. Beware that this `inlist` value isn't guaranteed to be correct, see [inlist issue](../ItemList/inlist%20issue.md) for more details.

##### Normal sub-handler

This sub-handler happens if `listredirect` isn't -2 or -3 (either gets handled by the other sub handler). It performs a redirect using an [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue line where the [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) is `listredirect` prepanded with a |[blank](Individual%20commands/Blank.md)\|. From there, processing resumes at the start of this new input string.

##### [additemtoss](Individual%20commands/Additemtoss.md) Special sub-handler

There is a special case if `listredirect` is -2 or -3 which should only happens through the [pickitem](Individual%20commands/Pickitem.md) that comes from a toss prompt from an [additemtoss](Individual%20commands/Additemtoss.md) command. For these 2 values, the handler is the following (for context, by this point, the caller should be an [Items](../Enums%20and%20IDs/Items.md), [flagvar](../Flags%20arrays/flagvar.md) 1 is the selected item if applicable and [flagvar](../Flags%20arrays/flagvar.md) 0 is the tossed item):

* On the caller, enable its gravity and its ccol, freeze all rotations on is rigid, set its bounces to 0 and child it to the current map.
* Start a LateVelocity (velocity applied after a frame) to RandomItemBounce(6.0, 12.0). This will make the item toss in a random direction.
* Destroys the first child of the caller's entity's sprite
* Force unfix the caller's entity, apply a touchcooldown on it of 70 frames, set its `tossed` to true and is timer to 600 frames (which is the time it takes before the item disappears)
* If `listredirect` is -2 (The toss was confirmed to swap the item, -3 has this part omitted as it means the toss was cancelled which means to not swap the item)
    * Set the caller's entity's animstate, itemstate and basestate to [flagvar](../Flags%20arrays/flagvar.md) 1 (this changes what the caller's item is to the swapped one)
    * Remove the item from the inventory matching the `listtype` used whose id is in [flagvar](../Flags%20arrays/flagvar.md) 1
    * Add the item to the inventory matching the `listtype` used whose id is in [flagvar](../Flags%20arrays/flagvar.md) 0
* Set [end](Individual%20commands/End.md) to true
* Reset `listredirect` to null
* Apply an `actioncooldown` of 30 frames
* If `eventtoss` is above -1, set `eventcall` to it

## Dialogue Cleanup

This phase only occurs if we are in [Dialogue mode](Dialogue%20mode.md). It tries to mostly do the reverse of dialogue setup, but with some added logic.

The first thing it will do is If we are using a [transfer](Individual%20commands/Transfer.md) or a [Warp](Individual%20commands/Warp.md) command and the current caller isn't null, set the caller.interactcd to 15.0. After, it will set the [tailtarget](Notable%20states.md#tailtarget) to stop talking if it isn't null and remove the [Text advance](Related%20Systems/Text%20advance.md)'s `skiptext`.

After, it will grab the [waitinput](Notable%20states.md#waitinput) lock and yield until it goes to false. One exception to this is if an [end](Individual%20commands/End.md) was processed since it serve as a bypass to this. In either case, the lock is released after as well as the [message](Notable%20states.md#message) lock.

From there, it's mostly cleaning up:

* Stops looping the bleep if it was playing and fade its sound to silence.
* Free all the letter slots of the [textholder](Notable%20states.md#textholder) and destroy it.
* The `minipause` lock is released If [event](Individual%20commands/Event.md)'s `inevent` is false and we are not in an actual event.
* Restores the [camoffset](Individual%20commands/Camoffset.md) to its original value if it was changed in dialogue setup.
* `overridefollower` is restored to its original value or to true if [Event](Individual%20commands/Event.md)'s `inevent` is true.
* If the [textbox](Notable%20states.md#textbox) isn't null, hide it with a shrink animation and Destroys the textbox in 1 second.

Special cleanup logic if the player isn't null and we aren't in an event:

- Restores the player rigid body constraints to its original value.
- Apply an `actioncooldown` of 20 frames
- Stops ignoring every collisions between the player's ccol and each entities that were ignored using any [igcolmove](Individual%20commands/Igcolmove.md) commands.

The last step of this phase is If we are not in an event and battle is null, call `EndOfMessage` which does some final logic:

* if [flag](../Flags%20arrays/flags.md) 347 (start of ch6) is true, sets the corresponding badgeshop's all bought flags to true (from 587 to 588) if the corresponding shop has 0 medals in stock. NOTE: 589 corresponds to an unused medal shop.
* If [flag](../Flags%20arrays/flags.md) 470 (Examined the view of Termite Capitol for the first time at the Forsaken Lands) is true and the Termite Kingdom discovery isn't unlocked, unlock it (TODO: recheck this and determine why?)
* if [flag](../Flags%20arrays/flags.md) 351 (Talked to the person in front of the Termacade for the first time) is true and the Termacade  discovery isn't unlocked, unlock it. TODO: recheck this
* if [flag](../Flags%20arrays/flags.md) 605 (Completed the Lost Books quest) is true and the Lost Book quests isn't in the completed array of [boardQuests](../Enums%20and%20IDs/BoardQuests.md), complete it.
* if the map isn't null, sets map.`currentline` to -1
* Check the achievements completion.

## Cleanup

This phase happens at the end of every SetText call. This is where some cleanups logic are performed, but also some [Commands](Commands.md) that needs some processing done at the very end rather than during the char loop:

* If we are in a [minibubble](Individual%20commands/Minibubble.md) command's inner call, the minibubble is destroyed. 
* A frame is yielded here.
* This is where the [warp](Individual%20commands/Warp.md) or [transfer](Individual%20commands/Transfer.md) are started if applicable.
* Something with [additemtoss](Individual%20commands/Additemtoss.md) ??? TODO: recheck this
* If this call had a [showtokens](Individual%20commands/Showtokens.md) command, the Game Token HUD element is destroyed.
* If for some reasons, there was a Game Token in key [Items](../Enums%20and%20IDs/Items.md), it is removed there.
* If the caller was an [Items](../Enums%20and%20IDs/Items.md) object, it is destroyed there.

Finally, [flag](../Flags%20arrays/flags.md) 349 is set to false if [ItemList](../ItemList/ItemList.md)'s `inlist` (check [inlist issue](../ItemList/inlist%20issue.md)) is false and we weren't in a [Prompt](Individual%20commands/Prompt.md), [NumberPrompt](Individual%20commands/NumberPrompt.md) or [LetterPrompt](Individual%20commands/LetterPrompt.md).
