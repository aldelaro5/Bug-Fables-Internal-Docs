# [SetText](SetText.md) Commands

A `Commands` is a enum used by [SetText](SetText.md) that indicates a command to processes something with or without parameters. When specifying the input string, a command and its parameters must be enclosed in `|`. The command name and each of its parameters must be separated by `,`. The command text is refered to anything between both `|` which the command has the ability to replace the text and resume processing at the same position.

Commands also has the ability to perform what's called a redirection which overwrites the input string with a new one and then resumes processing at the start of this new string.

## About case sensitivity

While the actual parsing of the command's name is case insensitive, it is strongly recommended to specify it in all lowercase. This is because there are some logic which are only compatible with the name being in all lowercase notably [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) and some checks such as the commands allowed in [testdiag](Individual%20commands/Testdiag.md) mode. Not putting it in all lowercase can result in commands being misidentified in some special cases.

For that reason, it is best to consider everything to be case sensitive unless specified otherwise.

## About the notes column

The notes contain useful information about each commands, here are what they mean:

* NOT REFERENCED: The enum value exists, but no implementation or references exists, the command name can be parsed without errors, but it will do nothing and so, it does not have a documentation page.
* UNUSED: The command remains functional and has an implementation, but there are no known usage in the game. Whether the command is useful or not can be found in its details page.
* DIALOGUE MODE: The command requires [Dialogue mode](Dialogue%20mode.md) or works incorrectly in [non Dialogue mode](Dialogue%20mode.md#non-dialogue-mode).
* REGULAR LETTER RENDERING: The command requires to be using [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) or works incorrectly in [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).
* SINGLE LETTER RENDERING: The command requires to be using [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) or works incorrectly in [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md).
* TESTDIAG: The command is supported by [Testdiag](Individual%20commands/Testdiag.md).
* BACKTRACK: The command is supported by the [Backtracking](Related%20Systems/Backtracking.md) system and its command text may be accumulated or it may directly interact with the system.
* REDIRECT: The command may cause a redirect and resume processing at the start of the new input string.
* REPLACE: The command may replace the command text and resume processing at the same position.
* PROCESSED IN ORGANISELINES: The command can be processed by [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) instead of [SetText](SetText.md).
* INFLUENCES ORGANISELINES: The command may influence the logic of [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) without necessarily being processed by it.

## About the enum's backing value

The actual int form of the enum isn't used in any way. Only the enum values's name is used for parsing. This means the backing value isn't relevant and as such, the table presented below will not show them in enum value order, but rather by logical categories. The ID column will show the value still for the sake of completeness. If you wish to only view the commands list sorted by their enum value, consult [this page](Commands%20sorted%20by%20enum%20value.md)

## Commands

There are 217 commands as of 1.1.2. Here are the different commands grouped by categories.

### Line

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|6|[Line](Individual%20commands/Line.md)|Go to a new line below and continue rendering at the start of the new line|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|14|[Halfline](Individual%20commands/Halfline.md)|An alias of `line` where `linespacing` is 0.5|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|87|[Quarterline](Individual%20commands/Quarterline.md)|An alias of `line` where `linespacing` is 0.25|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|115|[Pauseline](Individual%20commands/Pauseline.md)|An alias of `line` that is only processed by SetText when the pause menu is active and either in no window or in the following windows: top level window, "Inventory" and "Medals and Stats"|INFLUENCES ORGANISELINES, BACKTRACK|
|119|[Libraryline](Individual%20commands/Libraryline.md)|An alias of `line` that is only processed by SetText when the pause menu is active and the active window is "library"|INFLUENCES ORGANISELINES, BACKTRACK|
|120|[Shopline](Individual%20commands/Shopline.md)|An alias of `line` that is only processed by SetText when unpaused and the player has control or an [ItemList](../ItemList/ItemList.md) is getting processed|INFLUENCES ORGANISELINES, BACKTRACK|
|145|[Backline](Individual%20commands/Backline.md)|Does the same task than `line`, but with a different syntax and it only works in [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) while `line` works for both letter rendering method|REGULAR RENDERING|
|167|[Unpauseline](Individual%20commands/Unpauseline.md)|An alias of `line` that is only processed by SetText when the pause menu is not active|INFLUENCES ORGANISELINES, BACKTRACK|

### Size

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|67|[size](Individual%20commands/size.md)|Set the text size scaling to render from now on with the ability to prevent processing of further size commands temporarily|INFLUENCE ORGANISELINES, BACKTRACK|
|173|[Librarysize](Individual%20commands/Librarysize.md)|An alias of `size` where the command is only processed if the pause menu is active on the "Library" window|INFLUENCE ORGANISELINES, BACKTRACK|
|193|[Unpausesize](Individual%20commands/Unpausesize.md)|An alias of `size` where the command is only processed if the pause menu and the `questboardobj` for the [Quest Board List Type](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) are inactive|INFLUENCE ORGANISELINES, BACKTRACK|
|203|[Questsize](Individual%20commands/Questsize.md)|An alias of `size` where the command is only processed if the `questboardobj` for the [Quest Board List Type](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) is active|INFLUENCE ORGANISELINES, BACKTRACK|
|205|[Mapsize](Individual%20commands/Mapsize.md)|An alias of `size` where the command is only processed if the pause menu is active on the Map of Bugaria window|INFLUENCE ORGANISELINES, BACKTRACK|
|209|[Sizemulti](Individual%20commands/Sizemulti.md)|An alias of `size` in its second syntax|INFLUENCE ORGANISELINES, BACKTRACK|
|213|[Battlesize](Individual%20commands/Battlesize.md)|An alias of `size` or `sizemulti` that is only processed if a battle is in progress|BACKTRACK|
|214|[Pausesize](Individual%20commands/Pausesize.md)|An alias of `size` where the command is only processed if the pause menu is active and the questboardobj for the [Quest Board List Type](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) is inactive|INFLUENCE ORGANISELINES, BACKTRACK|
|216|[Listsize](Individual%20commands/Listsize.md)|An alias of `size` or `sizemulti` that is only processed if a battle is in progress, the pause menu is inactive and we are in an [ItemList](../ItemList/ItemList.md)|INFLUENCE ORGANISELINES, BACKTRACK|

### Flag arrays management

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|0|[String](Individual%20commands/String.md)|Replaces the text from this command to a [flagstring](../Flags%20arrays/flagstring.md) text with horizontal size clamping support|REPLACE|
|1|[Var](Individual%20commands/Var.md)|Replaces the text from this command to the textual representation of a [flagvar](../Flags%20arrays/flagvar.md) with left padding support or set a [flagvar](../Flags%20arrays/flagvar.md) to a value|REPLACE|
|33|[Flag](Individual%20commands/Flag.md)|Set a specific [flag](../Flags%20arrays/flags.md) slot or a one whose id is in a [flagvar](../Flags%20arrays/flagvar.md) to a value||
|66|[Discovery](Individual%20commands/Discovery.md)|Enable a [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) or enable all [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md), [Bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md), [Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md), [Records entry](../Enums%20and%20IDs/librarystuff/Records%20entry.md) and seen [Areas](../Enums%20and%20IDs/librarystuff/Areas.md)||
|78|[Regionalflag](Individual%20commands/Regionalflag.md)|Set a [Regionalflag](../Flags%20arrays/Regionalflags.md) slot to a value||
|92|[Flagvalue](Individual%20commands/Flagvalue.md)|Replace the text of this command by the string representation of a [flag](../Flags%20arrays/flags.md) slot or one where the slot is contained in a [flagvar](../Flags%20arrays/flagvar.md) slot|REPLACE|
|100|[Setvar](Individual%20commands/Setvar.md)|Set a [flagvar](../Flags%20arrays/flagvar.md) to a specific value, or increase/decrease a [flagvar](../Flags%20arrays/flagvar.md) by a value or another [flagvar](../Flags%20arrays/flagvar.md)||
|114|[Clonestring](Individual%20commands/Clonestring.md)|Copy the value of a [flagstring](../Flags%20arrays/flagstring.md) to another||
|127|[Resetregion](Individual%20commands/Resetregion.md)|Reset all [Regionalflags](../Flags%20arrays/Regionalflags.md) slot to false||
|128|[Mapflag](Individual%20commands/Mapflag.md)|Changes a mapflags slot using a specific id directly or one coming from a [flagvar](../Flags%20arrays/flagvar.md) slot||
|133|[Optiontovar](Individual%20commands/Optiontovar.md)|Set [flagvar](../Flags%20arrays/flagvar.md) 0 or a specific slot to the last selected option from a `prompt`, `numberprompt` or [ItemList](../ItemList/ItemList.md)||
|154|[Copyvar](Individual%20commands/Copyvar.md)|Set the value of a [flagvar](../Flags%20arrays/flagvar.md) slot|UNUSED|
|155|[Addvar](Individual%20commands/Addvar.md)|Perform an arithmetic operation on a [flagvar](../Flags%20arrays/flagvar.md) slot using a value as the other operand where the value is directly specified, corresponds to the current amount of berries or contained in another [flagvar](../Flags%20arrays/flagvar.md) slot||
|168|[Cberrytotal](Individual%20commands/Cberrytotal.md)|Replace the text of this command by the amount of [crystalbfflags](../Enums%20and%20IDs/crystalbfflags.md) that are set to true|REPLACE|
|185|[Itemname](Individual%20commands/Itemname.md)|Set a [flagstring](../Flags%20arrays/flagstring.md) to an [Item](../Enums%20and%20IDs/Items.md) or [Medal](../Enums%20and%20IDs/Medal.md)'s name where the id is specified directly or contained in a [flagvar](../Flags%20arrays/flagvar.md)||
|208|[Itemvalue](Individual%20commands/Itemvalue.md)|Set a [flagvar](../Flags%20arrays/flagvar.md) to an [Items](../Enums%20and%20IDs/Items.md) or [Medal](../Enums%20and%20IDs/Medal.md)'s value (buying price) where the id is specified directly or contained in a [flagvar](../Flags%20arrays/flagvar.md)||

### Dynamic text insertion

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|2|[Anstring](Individual%20commands/Anstring.md)|Replaces the text from this command to a [medal](../Enums%20and%20IDs/Medal.md) or [item](../Enums%20and%20IDs/Items.md)'s prepender string from the game data|REPLACE|
|40|[Currency](Individual%20commands/Currency.md)|Replaces this command's text with a textual representation of a number followed by a space followed by either the singular form of the game's currency (berry in English) or the plural form (berries in English) depending on a specific amount or an amount contained in a [flagvar](../Flags%20arrays/flagvar.md)|REPLACE|
|46|[Getstorage](Individual%20commands/Getstorage.md)|Replace the text of this command by the remaining amount of free [items](../Enums%20and%20IDs/Items.md) space of the storage (this is 35 - amount of items in storage under normal gameplay)|REPLACE|
|58|[Common](Individual%20commands/Common.md)|Replace the input string to a parameterless blank command followed by a line in commondialogue and continue processing at the start of it|REPLACE|
|138|[Sstring](Individual%20commands/Sstring.md)|Alias to `string` if processed by SetText directly, but it can also be processed before on [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) by replacing the text of this command to a [flagstring](../Flags%20arrays/flagstring.md)|PROCESSED IN ORGANISELINES|
|137|[Area](Individual%20commands/Area.md)|Replace the text of this command by the current [Area](../Enums%20and%20IDs/librarystuff/Areas.md)'s name|REPLACE|
|139|[Menu](Individual%20commands/Menu.md)|Alias to `string` if processed by SetText directly where `flagstring` is a menu text line id instead, but it can also be processed before by [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) with different behaviours|PROCESSED IN ORGANISELINES|
|157|[Call](Individual%20commands/Call.md)|An alias of `string` where `flagstring` is a [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) instead|REPLACE, TESTDIAG|
|169|[Medaltotal](Individual%20commands/Medaltotal.md)|Replace the text of this command by the amount of [Medal](../Enums%20and%20IDs/Medal.md) in the player's possession|REPLACE|
|178|[Lore](Individual%20commands/Lore.md)|A helper of `string` where [flagstring](../Flags%20arrays/flagstring.md) 0 is set to the Lore Book's text whose id is in [flagvar](../Flags%20arrays/flagvar.md) 0 with additional commands then the same [flagstring](../Flags%20arrays/flagstring.md) is sent to `string` in its fourth syntax|REPLACE|
|212|[Maxmedals](Individual%20commands/Maxmedals.md)|Replace this command's text by the number of collectible medals in the game (hardcoded to 108)|REPLACE|
|215|[GetFromMap](Individual%20commands/GetFromMap.md)|Replace the text of this command to a dialogue line that belongs to a specific [Map](../Enums%20and%20IDs/Maps.md)|REPLACE|
|217|[Plural](Individual%20commands/Layer.md)|Replaces this command's text by one of 2 strings depending on if a value or a [flagvar](../Flags%20arrays/flagvar.md) slot containing it is equal to 1 or not|REPLACE, UNUSED|

### Sprite rendering

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|47|[Button](Individual%20commands/Button.md)|Renders a ButtonSprite inline of the text and adjust the offset accordingly to resume processing after the sprite|REGULAR RENDERING, INFLUENCE ORGANISELINES, BACKTRACK|
|97|[Stars](Individual%20commands/Stars.md)|Renders a specific amount of white stars sprites in a row|REGULAR RENDERING|
|111|[Icon](Individual%20commands/Icon.md)|Render a gui sprite with a configurable size multiplier and sort value|REGULAR RENDERING, BACKTRACK, TESTDIAG|
|174|[Mothfly](Individual%20commands/Mothfly.md)|Replace the text of this command to a series of a specific amount of \|icon,146,`size`\| commands in a row where `size` is configurable|REPLACE|

### Conditional rediction and flow

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|34|[Checkmoney](Individual%20commands/Checkmoney.md)|Check that the berry count is less than a specific value or a value contained in a [flagvar](../Flags%20arrays/flagvar.md). If it is, redirect to another dialogue line|REDIRECT|
|35|[Checkinvqtd](Individual%20commands/Checkinvqtd.md)|Check that the quantity of [items](../Enums%20and%20IDs/Items.md) in an inventory is equal or higher than a specific quantity or the maximum amount allowed by the game for this specific inventory. If it is, replaces the input string by another dialogue line|REDIRECT|
|39|[Goto](Individual%20commands/Goto.md)|Redirect to a particular dialogue line or a randomly selected one from a list. This has support for appending commands and to not blank the textbox before the replacement|REDIRECT, BACKTRACK, TESTDIAG|
|44|[Checktrue](Individual%20commands/Checktrue.md)|Check a [flagvar](../Flags%20arrays/flagvar.md) for NOT being a specific integer, a single [flag](../Flags%20arrays/flags.md) for being true or a set of [flags](../Flags%20arrays/flags.md) for a specific state and either do nothing if all conditions are satisfied or redirect to a different line if they aren't|REDIRECT|
|59|[Checkvar](Individual%20commands/Checkvar.md)|Compare a [flagvar](../Flags%20arrays/flagvar.md) against a value or another [flagvar](../Flags%20arrays/flagvar.md) for a condition and redirect if this condition ends up being true or do nothing if it ends up being false|REDIRECT|
|76|[Checkregional](Individual%20commands/Checkregional.md)|Redirect to another dialogue line using a [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) if a [Regionalflag](../Flags%20arrays/Regionalflags.md) slot is false or if it is in a specific state|REDIRECT|
|77|[Checkflag](Individual%20commands/Checkflag.md)|A variation of `checktrue` where the only difference is the [flag](../Flags%20arrays/flags.md) condition are inverted|REDIRECT|
|129|[Checkmapflag](Individual%20commands/Checkmapflag.md)|Redirect to a [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) if a specific mapflags slot is true or continue processing as normal if it is false|REDIRECT|
|143|[Checkpos](Individual%20commands/Checkpos.md)|Redirect to a different line by a [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) if the current player's position on a specific axis isn't at least a specific value or do nothing otherwise|REDIRECT|
|148|[Battlewon](Individual%20commands/Battlewon.md)|Do nothing if the last battle or card battle was won, but redirect to a different dialogue line if it was lost or fled from|REDIRECT|
|163|[Checkanim](Individual%20commands/Checkanim.md)|Redirect to a different dialogue line if an [Entity](../Entities/Entity.md)'s [animstate](../Entities/EntityControl/Animations/animstate.md) is a specific value|REDIRECT|
|191|[Termacadecheck](Individual%20commands/Termacadecheck.md)|A `goto` helper where the [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) can be one of 2 values depending if both high scores at the Termacade games were beaten or not from their default one|REDIRECT, BACKTRACK|
|198|[Checkallquests](Individual%20commands/Checkallquests.md)|Redirect if 60 or more [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) are in the completed board or do nothing otherwise|REDIRECT|
|202|[Checksum](Individual%20commands/Checksum.md)|Process a `goto` in its first syntax if 2 values or [flagvar](../Flags%20arrays/flagvar.md) slots added together is strictly bigger than another value or [flagvar](../Flags%20arrays/flagvar.md) slot or do nothing otherwise|REDIRECT, BACKTRACK|
|206|[Caravanmedal](Individual%20commands/Caravanmedal.md)|Process a `goto` in syntax (2) if there are more than 1 prize [Medal](../Enums%20and%20IDs/Medal.md) set to be available through the Caravan shop or do nothing otherwise|REDIRECT, BACKTRACK|

### Dialogue flow

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|7|[Next](Individual%20commands/Next.md)|Signals the game that the current textbox is over and wait for a confirmation input before resuming with a fresh textbox|BACKTRACK, TESTDIAG, INFLUENCES ORGANISELINES|
|8|[End](Individual%20commands/End.md)|Signal to SetText to not wait for a confirmation input at the end after the input string has been fully processed|DIALOGUE MODE|
|9|[Break](Individual%20commands/Break.md)|Wait for a confirmation input before resuming||
|10|[Blank](Individual%20commands/Blank.md)|Clears the current textbox by freeing all letter slots and go at the start of the first line|BACKTRACK, INFLUENCES ORGANISELINES|
|11|[Lock](Individual%20commands/Lock.md)|Force a grab on the [message](Notable%20states.md#message) and the `minipause` locks|UNUSED, DIALOGUE MODE|
|15|[Stopskip](Individual%20commands/Stopskip.md)|Forces a stop on the text skip if one was in progress|DIALOGUE MODE|
|16|[Noskip](Individual%20commands/Noskip.md)|Forces a stop on the text skip if one was in progress and also prevents any text skip to occur from this point until it is allowed again|DIALOGUE MODE|
|25|[Spd](Individual%20commands/Spd.md)|Set the wait time in seconds between letters||
|26|[Speed](Individual%20commands/Speed.md)|An alias of `spd`||
|31|[Parent](Individual%20commands/Parent.md)|Change the caller to an [Entity](../Entities/Entity.md)'s [NPCControl](../Entities/NPCControl/NPCControl.md)||
|32|[Tail](Individual%20commands/Tail.md)|Set the [tailtarget](Notable%20states.md#tailtarget) to an [Entity](../Entities/Entity.md) and optionally change the animation state of the new target [Entity](../Entities/Entity.md)  with a hide/unhide of the textbox|DIALOGUE MODE, BACKTRACK, TESTDIAG|
|42|[Boxstyle](Individual%20commands/Boxstyle.md)|Sets the textbox's sprite or disable its rendering from now on and optionally specify its sortingOrder|DIALOGUE MODE|
|50|[Forcewait](Individual%20commands/Forcewait.md)|Yields for a certain amount of seconds if unpaused and stop a [Text advance](Related%20Systems/Text%20advance.md) `skiptext` if one was in progress||
|51|[Wait](Individual%20commands/Wait.md)|Yields for a certain amount of seconds if a [Text advance](Related%20Systems/Text%20advance.md) `skiptext` isn't in progress||
|60|[Fwait](Individual%20commands/Fwait.md)|An alias of `forcewait`||
|99|[Tailextra](Individual%20commands/Tailextra.md)|An alias of `tail` where the int form of `entity` refers to a temporary follower index instead of an [Entity id](Common%20commands%20id%20schemes/Entity%20id.md)|DIALOGUE MODE, BACKTRACK, TESTDIAG|
|101|[Gettail](Individual%20commands/Gettail.md)|An alias of `tail` where the int form of `entity` refers to the [AnimID](../Enums%20and%20IDs/AnimIDs.md) of the entity to set the [tailtarget](Notable%20states.md#tailtarget) instead of its [Entity id](Common%20commands%20id%20schemes/Entity%20id.md)|DIALOGUE MODE, BACKTRACK|
|104|[Hidespeed](Individual%20commands/Hidespeed.md)|Set the rate of the shrink/grow animation of the [textbox](Notable%20states.md#textbox) when it is hidden/revealed|DIALOGUE MODE|
|105|[Breakflag](Individual%20commands/Breakflag.md)|Set `end` to true if a specific [flag](../Flags%20arrays/flags.md) slot is false|UNUSED|
|106|[Bleep](Individual%20commands/Bleep.md)|Change the current dialogue bleep sound and its pitch from an [AnimID](../Enums%20and%20IDs/AnimIDs.md) data, [Entity](../Entities/Entity.md) field or specified directly|DIALOGUE MODE|
|147|[Boxspeed](Individual%20commands/Boxspeed.md)|An alias of `hidespeed`|DIALOGUE MODE|
|151|[Breakend](Individual%20commands/Breakend.md)|Replace the text of this command by \|`break`\|\|`destroyminibubble`\|\|`end`\||TESTDIAG|
|165|[Waitcn](Individual%20commands/Waitcn.md)|Yield until the chapter intro coroutine is over|UNUSED|
|166|[Faketail](Individual%20commands/Faketail.md)|Hide/shrink then reveal/grow the current textbox with a 0.15 seconds yield between after either operation|TESTDIAG|
|177|[Testdiag](Individual%20commands/Testdiag.md)|Enables the dialogue test mode until the end of SetText processing|UNUSED|
|195|[Lockbacktrack](Individual%20commands/Lockbacktrack.md)|Prevent any [Backtracking](Related%20Systems/Backtracking.md) from happening during the rest of the SetText call|BACKTRACK|

### [Entity](../Entities/Entity.md) control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|23|[Overfollower](Individual%20commands/Overfollower.md)|Toggles or set an override on followers to stop following their followee||
|28|[Anim](Individual%20commands/Anim.md)|Change the [animstate](../Entities/EntityControl/Animations/animstate.md) of the party or a specific [Entity](../Entities/Entity.md)||
|41|[Kill](Individual%20commands/Kill.md)|Kills an [Entity](../Entities/Entity.md) by calling [Death](../Entities/EntityControl/Notable%20methods/Death.md) (with `activatekill` true) on it||
|48|[Movewait](Individual%20commands/Movewait.md)|Yield until a specific [Entity](../Entities/Entity.md)'s forcemove lock is released||
|49|[Move](Individual%20commands/Move.md)|Instruct the game to start a forcemove on an [Entity](../Entities/Entity.md) to a specific position with a configurable speed, start and stop state||
|52|[Face](Individual%20commands/Face.md)|Instruct the party or an [Entity](../Entities/Entity.md) to face towards another one and optionally have that [Entity](../Entities/Entity.md) also face towards back||
|54|[Flip](Individual%20commands/Flip.md)|Toggle or set the `flip` state of an [Entity](../Entities/Entity.md) or the party||
|102|[Jump](Individual%20commands/Jump.md)|Make an [Entity](../Entities/Entity.md) jump up with a configurable velocity||
|116|[Addfollower](Individual%20commands/Addfollower.md)|Add the caller as a follower to a new [entity](../Entities/Entity.md) or add a new one via an [AnimID](../Enums%20and%20IDs/AnimIDs.md) or a [flagvar](../Flags%20arrays/flagvar.md) slot containing it||
|161|[Define](Individual%20commands/Define.md)|Add a name that can be used to resolve an [Entity id](Common%20commands%20id%20schemes/Entity%20id.md) for this SetText call|DIALOGUE MODE|
|179|[Follow](Individual%20commands/Follow.md)|An shorthand of `addfollower` in its second syntax||
|181|[Moveahead](Individual%20commands/Moveahead.md)|Instruct the game to start a forcemove on an [Entity](../Entities/Entity.md) where the destination is relative to its current position with a configurable speed, start and stop state||
|184|[Particle](Individual%20commands/Particle.md)|Play a particle effect at an [Entity](../Entities/Entity.md)'s position + a specific offset||
|187|[Entityalive](Individual%20commands/Entityalive.md)|Redirect if an [Entity id](Common%20commands%20id%20schemes/Entity%20id.md) resolves to not null and enabled, otherwise, this will not do anything and continue processing|REDIRECT|
|194|[Alwaysactive](Individual%20commands/Alwaysactive.md)|Set the `alwaysactive` field of an [Entity](../Entities/Entity.md) to a value||
|197|[Updateanim](Individual%20commands/Updateanim.md)|Call `UpdateAnimSepecifc` on an [Entity](../Entities/Entity.md)||
|200|[Deathsmoke](Individual%20commands/Deathsmoke.md)|Emit 20 particles at the parent or an [Entity](../Entities/Entity.md)'s position with configurable size||
|201|[Emoticon](Individual%20commands/Emoticon.md)|Show an emoticon above an [Entity](../Entities/Entity.md) for a given amount of frames||

### Party and player control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|12|[Cancelaction](Individual%20commands/Cancelaction.md)|Cancel the player's current field action|UNUSED|
|65|[Align](Individual%20commands/Align.md)|Move the party and Chompy's [Entity](../Entities/Entity.md) if present to be aligned in a row to the right or left of the caller via a forcemove||
|89|[Heal](Individual%20commands/Heal.md)|Heals the party fully or only heal the party's TP||
|90|[Lockmovement](Individual%20commands/Lockmovement.md)|Lock any rotation and movement to the player's entity's RigidBody with the exception of Y movement||
|91|[Teleportparty](Individual%20commands/Teleportparty.md)|Teleports the party's entities near the player's [Entity](../Entities/Entity.md) which optionally includes any temporary non party follower or teleport just the party entities at a specified distance and direction from the player's [Entity](../Entities/Entity.md)||
|110|[Exp](Individual%20commands/Exp.md)|Set, add or remove Exploration Points to the party by a value or by a value stored in a [flagvar](../Flags%20arrays/flagvar.md) slot||
|141|[Removestat](Individual%20commands/Removestat.md)|Remove a stat bonus by its index or a [flagvar](../Flags%20arrays/flagvar.md) slot containing it and optionally recalculate the party's stats after the removal||
|113|[Level](Individual%20commands/Level.md)|Set, add or remove ranks to the party by a value or by a value stored in a [flagvar](../Flags%20arrays/flagvar.md) slot||
|130|[Kinematicplayer](Individual%20commands/Kinematicplayer.md)|Set the `isKinematic` property of the player's entity's RigidBody to a value or turn it on temporarily for the rest of this SetText call||
|131|[Removefollower](Individual%20commands/Removefollower.md)|Remove a follower from the `extrafollowers` list by [AnimID](../Enums%20and%20IDs/AnimIDs.md) or remove all followers from it||
|142|[Addstat](Individual%20commands/Addstat.md)|Add a stat bonus by specifying its properties directly or by specifying [flagvar](../Flags%20arrays/flagvar.md) slots containing them and optionally recalculate the party's stats after the removal||
|158|[Igcolmove](Individual%20commands/Igcolmove.md)|Ignore all collision with the player to a specific [Entity](../Entities/Entity.md) during this SetText call|DIALOGUE MODE|
|196|[Fixchompy](Individual%20commands/Fixchompy.md)|Set Chompy's `following` to the last party's [Entity](../Entities/Entity.md)||

### Text rendering

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|13|[Center](Individual%20commands/Center.md)|Toggle the text centering during rendering||
|17|[Hide](Individual%20commands/Hide.md)|Hide or unhide the current textbox|DIALOGUE MODE|
|18|[Rainbow](Individual%20commands/Rainbow.md)|Toggles the rainbow [FontEffects](Related%20Systems/FontEffects.md) that will be effective from this point|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|19|[Shaky](Individual%20commands/Shaky.md)|Toggles the shaky [FontEffects](Related%20Systems/FontEffects.md) that will be effective from this point|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|20|[Wavy](Individual%20commands/Wavy.md)|Toggles the wavy [FontEffects](Related%20Systems/FontEffects.md) that will be effective from this point|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|21|[Glitchy](Individual%20commands/Glitchy.md)|Toggles the glitchy [FontEffects](Related%20Systems/FontEffects.md) that will be effective from this point|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|24|[Choicewave](Individual%20commands/Choicewave.md)|Enable a PromptAnim on the [textholder](Notable%20states.md#textholder) that will highlight the text with a wave visual effect if it is hovered on during a `prompt`||
|27|[Color](Individual%20commands/Color.md)|Set the text color to one of the game's text color palette for every letter from now on|REGULAR LETTER RENDERING, TESTDIAG|
|29|[Sort](Individual%20commands/Sort.md)|Changes the sorting order of every letter rendered from now on|REGULAR LETTER RENDERING|
|103|[Position](Individual%20commands/Position.md)|Set the vertical position of the [textholder](Notable%20states.md#textholder) relative to its parent|UNUSED|
|109|[Font](Individual%20commands/Font.md)|Change the current [fonttype](Notable%20states.md#fonttype) used by SetText from now on||
|112|[Dropshadow](Individual%20commands/Dropshadow.md)|Set the text's drop shadow offset vector or turn off the effect||
|117|[Fadeletter](Individual%20commands/Fadeletter.md)|Toggle the fade letter font effect from inactive to active or from active to inactive|REGULAR RENDERING|
|135|[Setbreak](Individual%20commands/Setbreak.md)|Changes the [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s `linebreak` value with the option to run [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) again after the change||
|144|[Triui](Individual%20commands/Triui.md)|Toggle the camera used to render each letters to 3DGUI instead of GUICamara (main camera in `tridimensional`)|REGULAR RENDERING|
|170|[Single](Individual%20commands/Single.md)|Toggle or set whether to render in [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)||
|171|[Tab](Individual%20commands/Tab.md)|Set the tabsize of the current letter slot in [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)|SINGLE LETTER RENDERING|
|172|[Singlebreak](Individual%20commands/Singlebreak.md)|Instruct [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) to use the specialized line breaks handling for [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md)|INFLUENCES ORGANISELINES, SINGLE LETTER RENDERING|
|176|[Backbox](Individual%20commands/Backbox.md)|Render a box behind the text with 40% transparency using one of the text colors||
|183|[Textangle](Individual%20commands/Textangle.md)|Reset or set the angles of the [textholder](Notable%20states.md#textholder)||
|190|[Limit](Individual%20commands/Limit.md)|Changes the [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s `linebreak` value|UNUSED, INFLUENCES ORGANISELINES|
|218|[Layer](Individual%20commands/Layer.md)|Changes the current layer to render from now on instead of the default layer which is 5 (`UI`)||

### Camera control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|53|[Camtarget](Individual%20commands/Camtarget.md)|Set the camera to target an [Entity](../Entities/Entity.md), target a specific position or remove any existing targets||
|82|[Camangle](Individual%20commands/Camangle.md)|Changes the main camera's angle||
|83|[Camoffset](Individual%20commands/Camoffset.md)|Changes the main camera's base position||
|86|[Resetcamera](Individual%20commands/Resetcamera.md)|Resets the camera to a default state||
|94|[Savecamera](Individual%20commands/Savecamera.md)|Saves the current camera's configuration and limits for retrieval later using `loadcamera`||
|95|[Loadcamera](Individual%20commands/Loadcamera.md)|Loads the saved camera's configuration and limits saved by `savecamera`||
|96|[Camspeed](Individual%20commands/Camspeed.md)|Set the camera's movement speed's multiplier||
|121|[Shakecamera](Individual%20commands/Shakecamera.md)|Shakes the camera's position by a configurable amount, time and whether or not to return to the initial position||
|122|[Removemaplimits](Individual%20commands/Removemaplimits.md)|Remove the position bound limits of the camera with the option to save the current ones to restore later using `resetmaplimits`||
|123|[Resetmaplimits](Individual%20commands/Resetmaplimits.md)|Restore the position bound limits of the camera from their original one when the map was loaded or from the ones saved using `removemaplimits` if they were saved to the temporary fields||
|164|[Camlimit](Individual%20commands/Camlimit.md)|Set the camera's positional lower bound limits (this command is likely broken, see its documentation for details)|UNUSED|

### Minibubble management

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|68|[Minibubble](Individual%20commands/Minibubble.md)|Show some text inside a Minibubble by a [Dialogue line id](Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or by directly specifying the text which will have its tail set to a specified entity. The Minibubble's lifecycle can then be controlled from the same SetText call using `waitminibubble` and `destroyminibubble`||
|69|[Destroyminibubble](Individual%20commands/Destroyminibubble.md)|Force the destruction of all or a specific `minibubble` by its index in the bubbles list if it wasn't destroyed yet||
|70|[Halt](Individual%20commands/Halt.md)|Completely stall SetText via an infinite loop that yields a frame (useful with `minibubble`)||
|71|[Waitminibubble](Individual%20commands/Waitminibubble.md)|Yield until all or a specific `minibubble` by its index in the bubbles list is destroyed||
|150|[Checkminibubble](Individual%20commands/Checkminibubble.md)|Redirect if any `minibubble` are present to a different dialogue line or do nothing if none are present|REDIRECT|

### Prompts

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|4|[Prompt](Individual%20commands/Prompt.md)|Display a prompt offering multiple options with cancellation support, then wait that an option has been chosen and process it in the same SetText call|DIALOGUE MODE, REDIRECT|
|43|[Pickitem](Individual%20commands/Pickitem.md)|Setup an [ItemList](../ItemList/ItemList.md) using [ShowItemList](../ItemList/ShowItemList.md) with the parameters sent which can get handled by SetText after the ItemList goes inactive|DIALOGUE MODE, REDIRECT|
|57|[NumberPrompt](Individual%20commands/NumberPrompt.md)|Displays a prompt where an integer number can be entered by entering each digits and redirects upon confirmation or cancellation. The result is stored in a [flagvar](../Flags%20arrays/flagvar.md) slot upon confirmation|DIALOGUE MODE, REDIRECT|
|160|[LetterPrompt](Individual%20commands/LetterPrompt.md)|Displays a prompt where a [flagstring](../Flags%20arrays/flagstring.md) can be edited by changing each letter and redirects upon confirmation|DIALOGUE MODE, REDIRECT|

### [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|80|[Addboard](Individual%20commands/Addboard.md)|Add a [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) to the open board|UNUSED|
|84|[Completequest](Individual%20commands/Completequest.md)|Sets a [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) as completed which removes it from the taken board and add it to the completed board||
|85|[Activateselectedquest](Individual%20commands/Activateselectedquest.md)|Mark a specific [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) or the one whose id is in [flagvar](../Flags%20arrays/flagvar.md) 0 as taken which removes it from the open board and adds it to the taken board||
|88|[Questprompt](Individual%20commands/Questprompt.md)|Sets the last `prompt` as a quest prompt which needs specific handling||
|182|[Addquest](Individual%20commands/Addquest.md)|Add a [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) to the open board or move an existing [BoardQuest](../Enums%20and%20IDs/BoardQuests.md) to another board||
|199|[Takeopenquests](Individual%20commands/Takeopenquests.md)|Move all [BoardQuests](../Enums%20and%20IDs/BoardQuests.md) in the open board to the taken board with the exception of the 5 bounty ones||
|204|[Questbreak](Individual%20commands/Questbreak.md)|Signal the char loop that processing is completed once this iteration ends under certain conditions (this command is broken, see its documentation section for details)|UNUSED|

### [Items](../Enums%20and%20IDs/Items.md), [Medals](../Enums%20and%20IDs/Medal.md) and currency control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|36|[Additemtoss](Individual%20commands/Additemtoss.md)|Give an [Item](../Enums%20and%20IDs/Items.md) or [medal](../Enums%20and%20IDs/Medal.md) based on its id or a [flagvar](../Flags%20arrays/flagvar.md) containing it. If it's an item and the item inventory is full, prompt to toss an item|REDIRECT|
|37|[Additem](Individual%20commands/Additem.md)|Give an [Item](../Enums%20and%20IDs/Items.md) based on its id or a [flagvar](../Flags%20arrays/flagvar.md) containing it or transfer all the items selected to or from the storage if the last [ItemList](../ItemList/ItemList.md) had a multi selection||
|38|[Money](Individual%20commands/Money.md)|Add or remove a certain amount of berries from the berries count which can optionally come from a [flagvar](../Flags%20arrays/flagvar.md)||
|45|[Removeitem](Individual%20commands/Removeitem.md)|Remove a [medal](../Enums%20and%20IDs/Medal.md) by its id or an [item](../Enums%20and%20IDs/Items.md) by its index from a specific inventory type where the item index or medal id is specified directly or is contained into a specified [flagvar](../Flags%20arrays/flagvar.md) slot. This can also remove all items that was multi selected in the last [ItemList](../ItemList/ItemList.md) if it was a `listsell`||
|79|[Createitem](Individual%20commands/Createitem.md)|Create an item [Entity](../Entities/Entity.md) of a specific [Item](../Enums%20and%20IDs/Items.md), [Medal](../Enums%20and%20IDs/Medal.md) or a Crystal Berry with configurable position and timer before disappearing||
|93|[Removebadgeshop](Individual%20commands/Removebadgeshop.md)|Removes a [medal](../Enums%20and%20IDs/Medal.md) from a medal shop by its [medal](../Enums%20and%20IDs/Medal.md) id or by a [flagvar](../Flags%20arrays/flagvar.md) containing it||
|98|[Giveitem](Individual%20commands/Giveitem.md)|Give an [item](../Enums%20and%20IDs/Items.md), [Medal](../Enums%20and%20IDs/Medal.md), Crystal Berry or berries to the player where its identification or amount is specified directly or is retrieved from a [flagvar](../Flags%20arrays/flagvar.md) with proper animations, presentations and checks that this action does not exceed the maximum amount of standard [items](../Enums%20and%20IDs/Items.md) if it is one|REDIRECT|
|118|[Setprize](Individual%20commands/Setprize.md)|Mark a Caravan prize medal as obtained via its [Medal](../Enums%20and%20IDs/Medal.md) id or the caller's [animstate](../Entities/EntityControl/Animations/animstate.md) which also removes it from the caravan list. This can also mark any prize medal as obtained via its prize id||
|132|[Shoppool](Individual%20commands/Shoppool.md)|Add a medal to a medal shop's pool by [Medal](../Enums%20and%20IDs/Medal.md) id or one coming from a [flagvar](../Flags%20arrays/flagvar.md), remove all medals from a specific shop's pool using a store id or one coming from a [flagvar](../Flags%20arrays/flagvar.md) or remove all medals from all shop pools||
|186|[Addprize](Individual%20commands/Addprize.md)|Mark a Caravan prize medal as available via its prize id||
|192|[Removeitemat](Individual%20commands/Removeitemat.md)|Remove an [Item](../Enums%20and%20IDs/Items.md) by its inventory type and its index in that inventory which may come from [ItemList](../ItemList/ItemList.md)'s `listoption`||
|211|[Rerollshops](Individual%20commands/Rerollshops.md)|Refresh all the shops and randomnise their available pool from their stock pool which changes which ones are on display||

### Transition and cutscenes

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|61|[FadeIn](Individual%20commands/FadeIn.md)|Play a fade in visual transition with options to specify the speed, color and the ability to destroy the caller after a `fadeout` done right after this one||
|62|[FadeOut](Individual%20commands/FadeOut.md)|Play a fade out visual transition with options to specify the speed after a `fadein`||
|74|[Innsleep](Individual%20commands/Innsleep.md)|Play the inn sleeping cutscene with configurable final positions and camera controls then yield until it is over||
|75|[Event](Individual%20commands/Event.md)|Start an [Event](../Enums%20and%20IDs/Events.md) or set if we are in an event or not||
|175|[Chapterintro](Individual%20commands/Chapterintro.md)|Play a chapter into and yield until it is done playing||
|180|[Transitionsort](Individual%20commands/Transitionsort.md)|Change the sortingOrder of the current sprite used for transitions||

### Battle

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|140|[Battle](Individual%20commands/Battle.md)|Starts an inescapable enemy battle with all or a specific list of [Enemies](../Enums%20and%20IDs/Enemies.md) with configurable battle map and music; then yield until the battle is over||
|146|[Cardbattle](Individual%20commands/Cardbattle.md)|Start a Spy Card battle with an opponent or the caller with a configurable map and deck; then yield until the battle is over||

### Termacade

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|152|[DungeonGame](Individual%20commands/DungeonGame.md)|Start a game of Mite Knight (aka MazeGame) and yield until it is over||
|153|[BeeGame](Individual%20commands/BeeGame.md)|Start a game of Flower Journey (aka FlappyBee) and yield until it is over||
|159|[PartyGame](Individual%20commands/PartyGame.md)|This command will cause an exception to be thrown (see its documentation for more details), do not use|UNUSED|
|188|[Scorecheck](Individual%20commands/Scorecheck.md)|Render the Termacade's games high scores and yield until Confirm or Cancel is pressed||

### Audio control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|124|[Music](Individual%20commands/Music.md)|Change the current music from its filename with a configurable fade speed or fade the current music to silence||
|125|[Sound](Individual%20commands/Sound.md)|Play a sound effect music from its file path with a configurable pitch and volume with the option to loop it at the first free AudioSource or a specific one by its id||
|189|[Fademusic](Individual%20commands/Fademusic.md)|Fade the music to silence with a configurable fading speed||

### HUD visibility

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|72|[Showmoney](Individual%20commands/Showmoney.md)|Show the currency count HUD element||
|73|[Hidemoney](Individual%20commands/Hidemoney.md)|Hide the currency count HUD element||
|156|[Showtokens](Individual%20commands/Showtokens.md)|Create or destroy a Game Tokens counter HUD element||

### Global game functions

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|30|[Save](Individual%20commands/Save.md)|Attempts to save the game and continue processing with an output message|REPLACE|
|107|[Exitgame](Individual%20commands/Exitgame.md)|Immediately stops the game's execution if called from the StartMenu's settings or reset the game's scene after a transition if called from the pause menu after loading a file|UNUSED|
|108|[Openpause](Individual%20commands/Openpause.md)|Change the active window of the existing pause menu||

### Map load

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|55|[Warp](Individual%20commands/Warp.md)|Setup a [map](../Enums%20and%20IDs/Maps.md) transfer to either a specific [map](../Enums%20and%20IDs/Maps.md) id or one that is contained in a [flagvar](../Flags%20arrays/flagvar.md) with the optional ability to set where to move the party after the new [map](../Enums%20and%20IDs/Maps.md) is loaded||
|56|[Transfer](Individual%20commands/Transfer.md)|An alias of `warp`||
|162|[Loadmap](Individual%20commands/Loadmap.md)|Reload the current [Map](../Enums%20and%20IDs/Maps.md) or load another one||

### [Char loop](Life%20Cycle.md#char-loop) control

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|210|[Ignorenext](Individual%20commands/Ignorenext.md)|Signal SetText to not process the next n commands where n is configurable|UNUSED|

### Miscellaneous

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|126|[Destroydescbox](Individual%20commands/Destroydescbox.md)|Destroy the caller's description window if it exist no matter what or only do it if the caller's interaction type isn't Shop or CaravanBadge||
|136|[Librarybook](Individual%20commands/Librarybook.md)|Signal the game that a Lore Book has been acquired and refresh the LibraryShelf||
|149|[Switch](Individual%20commands/Switch.md)|Toggle or set the hit field of an [Entity](../Entities/Entity.md)'s [NPCControl](../Entities/NPCControl/NPCControl.md) (this is meant to toggle or set the state of an [Entity](../Entities/Entity.md) of type switch)||

### Not referenced

|ID|Name|Summary|Notes|
|--:|-------|-------|-----|
|3|`Checkitem`|N/A|NOT REFERENCED|
|5|`Getitem`|N/A|NOT REFERENCED|
|22|`Buffer`|N/A|NOT REFERENCED|
|63|`LeafIn`|N/A|NOT REFERENCED|
|64|`LeafOut`|N/A|NOT REFERENCED|
|81|`Openboard`|N/A|NOT REFERENCED|
|134|`Librarybreak`|N/A (This is processed by PauseMenu)|NOT REFERENCED |
|207|`Name`|N/A|NOT REFERENCED|
