# Update and FixedUpdate
This pages talks about the Update and FixedUpdate Unity events of MainManager.

## Update
MainManager.Update is the main global update method of the game. It only does anything if `basicload` is true (LoadEverything completed).

Here is a summary of the tasks it handles in order:

- Handles [dialogue mode](../SetText/Dialogue%20mode.md) SetText updates when not in a `prompt` or `itemlist`:
    - Handles the `blinker` rendering when `waitinput` is true
    - Enables `isholdingskip` when input 5 (cancel) is held (check the [text advance system](../SetText/Related%20Systems/Text%20advance.md) documentation to learn more)
    - Handles passing to the next textbox (or going forward when [backtracking](../SetText/Related%20Systems/Backtracking.md)) when input 4 (confirm) or 5 (cancel) is pressed
    - Handles initiating a backtrack if input 6 (switch) is pressed
    - Handles stopping `isholdingskip` when input 5 (cancel) is no longer held
- Handles different types of prompts (check the [prompt](../SetText/Individual%20commands/Prompt.md), [numberprompt](../SetText/Individual%20commands/NumberPrompt.md) and [letterprompt](../SetText/Individual%20commands/LetterPrompt.md) documentations for the details, this only summarises what happens):
    - On all prompts:
        - The `blinker` is disabled
        - PlayScrollSound called when up/down inputs are processed
        - Both `keyhold` are reset to 0.0 when no longer holding any directional inputs
    - On a regular [prompt](../SetText/Individual%20commands/Prompt.md):
        - `option` decrements on an up input (with wrap around to `maxoption` - 1 when going negative) and increments on a down input (with wrap around to 0 when going to `maxoption`)
        - On a 4 input (confirm), the confirmation is handled with a `Confirm` sound, setting `listcancelled` to false, setting `promptpick` and `lastprompt` to the confirmed `option`, turning off the `prompt` flag and resetting the backtracking lines. This also turns any current `skiptext` and applies an `inputcooldown` of 5.0 frames
        - If `listcancel` isn't negative (this prompt supports cancellation) and input 5 (cancel) is pressed, it is handled with a `Cancel` sound, setting `promptpick` and `lastprompt` to `listcancel` and turning off the `prompt` flag. This also turns any current `skiptext` and applies an `inputcooldown` of 10.0 frames
    - On a [numberprompt](../SetText/Individual%20commands/NumberPrompt.md):
        - `option` is set to 0 on an up input and to `maxoption` - 1 on a down input. Either inputs causes [flagvar](../Flags%20arrays/flagvar.md) 5 to be set to the input id
        - When input 8 (pause) is pressed, `option` is set to `maxoption` - 1
        - `option` decrements on a left input (with wrap around to `maxoption` - 1 when going negative) and increments on a right input (with wrap around to 0 when going to `maxoption`). Either inputs causes [flagvar](../Flags%20arrays/flagvar.md) 5 to be set to the input id and PlayScrollSound to be called
        - On a 4 input (confirm), the same handling is done than a regular prompt except that `promptpick` is set to 0 and there are further handling depending on the `option` confirmed: anything less than 10 appends the number to [flagstring](../Flags%20arrays/flagstring.md) 0, `option` 10 either commits the flagstring to `flagvar[listtype]` or cancels it if it's empty and `option` 11 removes 1 character from the flagstring when it's not empty. On a confirm or cancel, `promptpointers` is set to one element being `listcancel` or `listredirect` for cancelling or confirming respectively. A PayBuzzer happens when attempting to erase a number when the flagstring is empty
        - On a 5 input (cancel), the same handling is done than a regular prompt except that `promptpick` is set to 0, `promptpointers` is set to `listcancel`, the `inputcooldown` is 15.0 frames instead and the backtracking is reset
    - On a [letterprompt](../SetText/Individual%20commands/LetterPrompt.md):
        - `option` is decreased by flagvar 1 on an up input (clamped from 0) and it is increased by flagvar 1 on a down input (clamped to `maxoption` - 1). Either inputs causes [flagvar](../Flags%20arrays/flagvar.md) 5 to be set to the input id
        - When input 8 (pause) is pressed, `option` is set to `maxoption` - 1
        - When input 6 (switch) is pressed, the letterprompt is changed to the next one with ChangeLetterPrompt(-1)
        - The same handling of left/right inputs happens than a numberprompt, but with the variation that `option` is wrapped to the leftmost of the current row by increasing it by flagvar 1 if left was pressed and `option` is wrapped on the rightmost of the current row by decreasing it by flagvar 1 if right was pressed
        - Any directional input processed above gets its changes to `option` repeated if the current `option` points to a blank space as stored by flagvar 5. `option` gets clamped from 0 to `maxoption` - 1 and flagvar 5 gets reset to -1 after
        - On a 4 input (confirm), the same handling is done than a regular prompt except that `promptpick` is set to 0 and there are further handling depending on the `option` confirmed: anything less than `maxoption` - 3 appends the selected character to `flagstring[listtype]` (with Koeran handling if needed), `option` of `maxoption` - 3 removes a character in the flagstring (with Koeran handling if needed), `option` of `maxoption` - 2 appends a ` ` to the flagstring and `option` of `maxoption` - 1 commits the flagstring to `flagvar[listtype]` and sets `promptpointers` to one element being `listredirect`. A PayBuzzer happens when attempting to erase a character when the flagstring is empty, when attempting to add a character while its length is 10, when attempting to add a space while in the middle of Building a Korean character or when attempting to confirm when the flagstring is empty (cancelling isn't supported on a letterprompt)
        - On a 5 input (cancel), either the last entered part of the current Korean character is undone or a character is removed from `flagstring[listtype]` (PlayBuzzer is called instead if the flagstring was empty)
- Handles specifics about cooking under specific conditions where `sounds[5]` and `sounds[6]` have their pitch set to 2.5, Time.TimeScale set to 2.5 and `noskip` set to true. Here are the list of required conditions for that to happen:
    - [flagvar](../Flags%20arrays/flagvar.md) 3 is 5353 (only set to this value by the cooking event when the chef is about to cook)
    - The game is `inevent`
    - The [message](../SetText/Notable%20states.md#message) lock is released
    - `noskip` is false (meaning the logic hasn't run yet)
    - `lastevent` is 3 (cooking event)
    - `skiptext` is true
    - `inputcooldown` expired
    - Either a 4 (confirm) input or 5 (cancel) input gets held
- Handles various [itemlist](../ItemList/ItemList.md) related naviguation (check the different [listtype](../ItemList/listtype.md) documentations for the details, this only summarises what happens):
    - If `listsell` is true, `showmoney` is set to 10.0 so the berry counter [HUD](../General%20systems/HUD.md) is shown
    - `cursor` gets locally positioned relative to `listcursor` so it visually points at the current item. There are slight variations for the [language listtype](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md) and any [quests listtype](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)
    - Up/down inputs gets processed with an [UpdateList](../ItemList/Utility%20methods.md#updatelist) call (true for up, false for down, 1 skip and without nosound) followed by a [ShowItemList](../ItemList/ShowItemList.md) with the current ItemList parameters to refresh `itemlist` if `option` changed as a result of the UpdateList call. NOTE: This doesn't use the overloads that supports the wrap around feature
    - Left/Right inputs gets processed the same way than up/down, but the UpdateList calls are repeated for `listammount` times before the ShowItemList call (it doesn't use the overloads that supports this already with the skip parameter). However, for a [quests listtype](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md), these inputs processing completely changes because it needs to cycle the current tab:
        - PlaySound(`PageFlip`) called
        - ResetList called
        - `listtype` is decremented for a left input (with wrap arround to 16 if it gets lower than 14) or incremented for a right input (with wrap aroundf to 14 if it gets higher than 16)
        - UpdateQuestBoard called which rerenders the now changed tab
        - ShowItemList called to reflect the new listtype with showdescription and without sell
    - If no directional inputs are pressed both `keyhold` are reset to 0.0
    - When pressing input 7 (show HUD) on an ItemList that supports `multiselect`, the `option` is added (if allowed) or removed from `multiselect` (with `BadgeEquip` or `BadgeDequip` sound) followed by `listY` set to -1 with a ShowItemList call using the current `itemlist` parameters to refresh the list if the changes to `multiselect` was allowed (PlayBuzzer call if it wasn't). Here are the conditions to support `multiselect` (one of them needs to be true):
        - `listsell` is true
        - `listtype` is [items listtype](../ItemList/List%20Types%20Group%20Details/Items%20List%20Type.md) while [flags](../Flags%20arrays/flags.md) 349 is true (this is a items storage deposit process). In this condition, it is forbidden to add to `multiselect` when it already contains enough to fill up `maxstorage` if it was confirmed
        - `listtype` is storage items listtype (assumed to be a withdrawal process). In this condition, it is forbidden to add to `multiselect` when it already contains enough to fill up `maxitems` if it was confirmed
    - When pressing input 4 (confirm), the following happens:
        - PlaySound(`Confirm`, 10) called
        - `listcancelled` set to false
        - `lastprompt` set to `option`
        - If `savelastlist` is enabled, it is set to false while `overridedlist` is set to the return of a SaveList
        - There are specific confirmation handling for most of the listtypes, check the [listtype](../ItemList/listtype.md) documentation for the details on each of them
    - When pressing input 5 (cancel), the following happens (except for a [language listtype](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md) which cannot be cancelled):
        - `multiselect` is reset to a new list
        - PlaySound(`Cancel`, 10) called
        - `listcancelled` set to true
        - `inputcooldown` set for 15.0 frames
        - `skiptext` reset to false
        - On a [quests listtype](../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md), CloseQuestBoard is called and SaveCameraPosition(false) is called to restore the save camera position (check the [quest board system](../General%20systems/Quest%20boards.md) to learn more)
        - If the above didn't apply and the game is in a `battle`, [CancelList](../Battle%20system/Player%20UI/CancelList.md) is called on the `battle`
        - If neither of the above cases applied, `listredirect` is set to `listcancel`
        - DestroyList called to tear down the current `itemlist`
- Calls GetJoystick TODO: detail when InputIO is documented

## FixedUpdate
MainManager.FixedUpdate is much simpler in comparision.

If `basicload` is true ([LoadEverything](Boot%20and%20reset%20process.md#loadeverything-part-12) ran to completion), the following happens:

- [RefreshCamera](../General%20systems/Camera%20system.md#refreshcamera) is called
- [LoopMusic](../General%20systems/Music%20playback.md#music-looping) is called
