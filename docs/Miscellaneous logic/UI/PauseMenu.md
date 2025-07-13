# PauseMenu
PauseMenu is the component that implements the pause menu of the game, but it can be used from the StartMenu (only the settings window) and there is a lite version available from a retryable battle when selecting the "change loadout and retry" option.

This component is very complex because most of its logic is extremely verbose UI rendering. Because of this, not much will be documented about its internals, but the parts that will be documented are useful for the rest of the game's logic.

## Pausing
Pausing in the overworld is entirely managed by PlayerControl's [GetInput](../PlayerControl/GetInput.md) when the pause input is pressed and the game is pausable. When that happens, a new GameObject is created named `PauseMenu` with the PauseMenu component on it.

This leads to its start which renders windows 0 (the main pause screen). More importantly, Start sets MainManager.`pausemenu` to the GameObject and instance.`pause` to true. This is the strongest pausing kind in the game, even stronger than a `minipause` and it will prevent a lot of logic to happen in the rest of the game.

As for how this is setup from the StartMenu, it's done by setting the `windowid` of the component to 4 (settings) after it was added and also setting its `calledfrommain` to true. This will cause some special handoff to happen when unpausing to go back to the title screen menu.

Finally, for the battle pausing, it goes the same as the overworld pausing, but PauseMenu itself will change its handling to not allow to go to some windows since MainManager.`battle` will exist by this point.

## Unity events
The section above brought up Start, but the component also uses other Unity events:

- OnGUI: Monitor [keyboards](../InputIO/Keyboard.md) inputs when rebinding it
- FixedUpdate: Changes the color of the `dimmer` (a child GameObject that controls how faded the game looks). The only thing it really does is to lerp its color to Color.clear when unpausing
- Update: The main event used for most of the UI handling logic of the menu

## Unpausing
Exiting the pause menu first involves the DestroyPause method which does some tasks such as update the Animator on the `playerdata` if needed (it's possible HARDEST was active and the equip state of `DoublePain` changed). It also updates the `playerdata` sprite materials to handle changes to the equip state of `HoloCloak`.

At the end, DestroyPause is called in 0.25 seconds which is what will do the actual unpausing. Here's what it does:

- Set instance.`pause` to false
- Destroys the current [ItemList](../ItemList/ItemList.md) if one is present
- [CheckAchievement](Achievements.md#checkachievement) is invoked in 0.5 seconds
- MainManager.`player`.`pausecooldown` is set to 7.0 frames
- The GameObject with the PauseMenu gets destroyed

## Windows
The pause menu is divided into multiple GUI windows and they have ids associated with them that's used and agreed upon by the rest of the game. Here's the mapping:

|Id|Description|
|-:|-----------|
|0|The main pause menu screen|
|1|Items inventory|
|2|Medals and skills|
|3|Library|
|4|Settings|
|5|Keyboard bindings|
|6|World map|
|7|An UNUSED controller bindings window, this was replaced with the CUSTOM BINDING wizard from the settings|
