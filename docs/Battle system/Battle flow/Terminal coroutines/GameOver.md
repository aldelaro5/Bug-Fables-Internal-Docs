# GameOver
This is a terminal coroutine that will present the player with the game over screen and a prompt to reload, return to main menu or if allowed, retry or retry with the ability to manipulate the standard items and medals using a mini PauseMenu.

```cs
private IEnumerator GameOver(bool skipsetup)
```

## Parameters

- `skipsetup`: If true, it will skip some setup logic at the start and only show the prompt immediately. This is meant only to be used recursively in an edge case where reloading the save file failed which lets the player remedy the problem before letting GameOver try again, but with less setups. This should NEVER be called with the value false for the first time because it will prevent to change to a [terminal flow](../Update.md#terminal-flow)

## Procedure

- `alreadyending` is set to true TODO: what is this
- [ItemList](../../../ItemList/ItemList.md)'s `listredirect` is set to -1 (this workarounds a potential [inlist issue](../../../ItemList/inlist%20issue.md) that can happen as a result of the [SetText](../../../SetText/SetText.md) call later)
- The [SetText](../../../SetText/SetText.md) string is prepared and it starts with `|`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,-1|` which renders without a dialogue box. This string will get appended when needed later

### Setup
This entire section is skipped if skipsetup is true.

- All `smallexporbs` and `bigexporbs` are destroyed
- `action` and `cancelupdate` are set to true changing to a [terminal flow](../Update.md#terminal-flow)
- [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called
- SetFlags is called which sets all [flags](../../../Flags%20arrays/flags.md) and [flagvar](../../../Flags%20arrays/flagvar.md) to the ones from the [StartupData](../../StartUpData.md) of the `sdata`
- instance.`hudcooldown` is set to -1 which hides the HP HUD elements
- FadeMusic is called with a speed of 0.035
- A RoundTransition is played via PlayTransition to pure black with a speed of 0.05 and a data of -10 (which is the sortingOrder)
- A frame is yielded
- instance.`transitionobj[0]` z local position is set to 15.0 and its SpriteRenderer's sortingOrder is set to -100
- 2.0 seconds are yielded
- The `cursor` is destroyed if it existed
- If the `GameOver` sound isn't playing, it is played (as `Gameover`, but it works still) and the audiosource's volume is set to MainManager.`musicvolume` which is needed because it is considered a sound instead of a music
- The [SetText](../../../SetText/SetText.md) string gets the following ammended to it: `|boxstyle,-1||center||line||line||line||line||line||halfline||spd,0||color,4||sort,10||fadeletter|` followed by `menutext[146]` (the game over text shown at the top) followed by `|`[fwait](../../../SetText/Individual%20commands/Fwait.md)`,4.5|`. This is basically the game over text positioned a bit down using empty lines and fading in for 4.5 seconds

### [SetText](../../../SetText/SetText.md) call
The following and any further section happens regardless of skipsetup.

- `actiontext` and `hexpcounter` are destroyed if they exist
- The SetText string gets its final append and it depends on the value of `canflee` (meaning the battle is escapable). To note, `menutext[78]` only contains `|`[end](../../../SetText/Individual%20commands/End.md)`|` because these prompt options are blanks: they don't cause SetText to continue since they will be handled after the call:
    - If it's true, `|`[prompt](../../../SetText/Individual%20commands/Prompt.md)`,menu,3,2,78,78,135,36,none|` is appended
    - If it's false, `|`[prompt](../../../SetText/Individual%20commands/Prompt.md)`,menu,4.5,4,78,78,78,78,134,136,135,36,none|` is appended
- The SetText call happens in [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) using the final string generated and the following properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of `BubblegumSans`
    - linebreak of `messagebreak`
    - No tidimensional
    - position of MainManager.`bubblepos` (normally (0.0, 4.15.0, 10.0))
    - No cameraoffset
    - size of Vector2.one
    - No parent
    - No caller

#### Broken failsafe logic
This is where a very broken failsafe logic occurs. See the note block below for details on why this logic should never happen.

From there, GameOver will yield all frames for a maximum of 330.0 frames until instance.`promptbox` exists which is when the prompt appears. This is tracked with a local frame countdown decremented by MainManager.`framestep` followed by a frame yield on each iteration. If the countdown reaches 0.0 before `promptbox` exists:
    
- `gameover` is set to a new GameOver call with skipsetup
- A yield break happens which abruply stops this coroutine

> NOTE: This failsafe was supposed to be a workaround to a previous [inlist issue](../../../ItemList/inlist%20issue.md), but this did not worked because crucically, it breaks the [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) rule because the [message](../../../SetText/Notable%20states.md#message) will still be grabbed if this failsafe triggers. This means the second GameOver call will allow SetText to process a dialogue mode call WHILE ONE IS ONGOING! This is extremely dangerous because it will lead to 2 SetText calls fighting for each other or even allow player control during a dialogue mode call because one of the prompt option reloads a save. This logic should NEVER trigger under any circumstances otherwise, it will break SetText and general game flow! The original inlist issue has been sucessfully workarounded by setting MainManager.`listredirect` to -1 earlier so it provides no benefit.

#### Proper [message](../../../SetText/Notable%20states.md#message) lock wait
After avoiding the broken failsafe (which happens due to instance.`promptbox` existing after 4.5 seconds instead of 5.5 seconds) instance.`promptbox` local z position is set to 10.0.

This is followed by all frames being yielded until the [message](../../../SetText/Notable%20states.md#message) lock gets released. This is a proper and safe wait for the dialogue mode call to finish unlike the broken failsafe above.

### Prompt option handling
The `GameOver` (as `Gameover` but it works anyway) sound is stopped with a speed of 0.002.

From there, the specific prompt options are handled by checking `canflee` and instance.`option`. 

If you can flee, there's 2 prompt options:

- 0: Reload the save
- 1: Reload the scene (this will reset the whole game and go back to the main menu)

If you can't flee, there's 4 prompt options (2 are common with the can flee case):

- 0: Retry
- 1: Open a pause menu allowing to manipulate standard items and medals before retrying
- 2: Reload the save
- 3: Reload the scene (this will reset the whole game and go back to the main menu)

The following details what each option's handling does. No matter which one is handled, a frame is yielded at the very end after the option handler is done.

#### Retry

This only calls [Retry](../Retry.md) without skipdata.

#### Change loadout and retry

- ReloadInitialData is called which does the following (mostly loads from the [StartUpData](../../StartUpData.md)):
    - [ApplyBadges](../../ApplyBadges.md) is called
    - [ApplyStatBonus](../../ApplyStatBonus.md) is called
    - instance.`maxtp` is set to `sdata.maxtp`
    - instance.`tp` is set to `sdata.tp` clamped from 0 to instance.`maxtp`
    - For each player party member:
        - `atk` is set to the matching `sdata.atk`
        - `def` is set to the matching `sdata.def`
        - `hp` is set to the matching `sdata.hp` clamped from 0 to the player party member's `maxhp`
        - `lockitems` is set to the matching `sdata.partyitemuse`
    - instance.`items[0]` (standard items) is set to `sdata.items`
    - instance.`items[1]` (key items) is set to `sdata.keyitem`
- instance.`inbattle` is set to false (this is only temporary, but needed to have the correct UI naviguation logic to use the PauseMenu)
- MainManager.`pausemenu` is set to a new GameObject named `PauseMenu` with a PauseMenu component added to it
- A frame is yielded
- [ApplyBadges](../../ApplyBadges.md) is called
- All frames are yielded while `pausemenu` exists
- [Retry](../Retry.md) with skipdata is called

#### Reload save
There is a special failsafe that occurs if the existing [save file](../../../External%20data%20format/Save%20File.md) no longer exists for some reasons. Here is what happens in that case:

- A frame is yielded
- A new [SetText](../../../SetText/SetText.md) call happens in [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the text being `|boxstyle,4||center||halfline||spd,0|` followed by `menutext[145]` which is an error message informing the player of the problem. The call also has these properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of `BubblegumSans`
    - linebreak of `messagebreak`
    - No tidimensional
    - position of Vector3.zero
    - No cameraoffset
    - size of Vector2.one
    - No parent
    - No caller
- All frames are yielded while the [message](../../../SetText/Notable%20states.md#message) lock is grabbed. This allows the player to possibly remedy the problem by moving the save file at the correct location or to load the backup one. Since no [end](../../../SetText/Individual%20commands/End.md) commands was processed here, there will be a `waitinput` at the end allowing this failsafe to be remediable
- `gameover` is set to a new GameOver call with skipsetup. This is safe unlike the failsafe because in this case, there are no SetText call running anymore so this will simply present the same prompt and the player is able to try the reload option again. Since this GameOver call finishes shortly after, it essentially transferts complete control to the new one and it allows this cycle to repeat over and over

If the save does exist, the following is done instead:

- 0.5 seconds are yielded
- MainManager.ReloadSave is called which does the following:
    - The `CrowdChatter` sound is stopped
    - FadeMusic is called with a speed of 0.1
    - All coroutines belonging to MainManager.`battle` are stopped and `battlemap` is destroyed
    - All coroutines belonging to MainManager.`events` (the EventControl) are stopped
    - All player party member's `entity` are destroyed
    - RenderSettings.skybox is set to null
    - MainManager.`map` is destroyed
    - [Event](../../../Enums%20and%20IDs/Events.md) 22 is started via `events` without a caller (this is the save loading event)
- instance.`inbattle` is set to false
- We enter a `minipause`
- `pause` is set to false
- This BattleControl is destroyed

#### Return to main menu

- The `CrowdChatter` sound is stopped
- 0.5 seconds are yielded
- SceneManager.LoadScene(0) is called which resets the entire game
