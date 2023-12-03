# QuestBoard
The interaction for a quest board which allows to consult the 3 quest boards and take any open quests by having a dialogue with the caretaker of the quest board which is another NPCControl.

## args meaning
None.

## Data meaning
- `data[0]`: An [entity id](../../../SetText/Common%20commands%20id%20schemes/Entity%20id.md) that corresponds to the caretaker of the quest board
- `data[1]`: A [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) of what the caretaker of the quest board will say when accepting to take a quest. The line must contain a [prompt](../../../SetText/Individual%20commands/Prompt.md) for the system to work because this line will get prepended with a [questprompt](../../../SetText/Individual%20commands/Questprompt.md) when confirming the quest. Check the documentation of that command to learn more about how this all work
- `data[2]`: If it's not negative, this is a [flag](../../../Flags%20arrays/flags.md) slot that if it's false, [Interact](../Interact.md) will be called on the `npcdata` of the map entity whose id is `data[0]`.
- `vectordata[0]`: The camera position to move before showing the quest board UI, set to instance.`camoffset`
- `vectordata[1]`: The camera angles to move before showing the quest board UI, set to instance.`camangleoffset`
- `vectordata[2].x`: The camera speed to usd while moving the camera before showing the quest board UI, set to instance.`camspeed`
- `vectordata[2].y`: The time in seconds to let the camera move before showing the quest board UI

## LateUpdate (SpecialTattles)
The `tattleid` is set to -1 ("L.", this isn't supposed to be tattled as this text is an internal error message).

## Interact
If `data[2]` isn't negative and its corresponding [flag](../../../Flags%20arrays/flags.md) slot is false, [Interact](../Interact.md) is called on the `npcdata` of the map entity whose id is `data[0]` without any arguments.

Otherwise, an OpenQuestBoard coroutine is started with the map entity whose id is `data[0]` as the caretaker and this as the caller.

## PlayerControl.LateUpdate
entity.`emoticonid` gets set to 1 (a blue ?) and entity.`emoticoncooldown` gets set to 2.0 if all of the following conditions are met:
- We aren't in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging`, `flying`, `startdig` or in `shield`
- This is the closest NPC in the `npc` list as of the last 3 frames

## MainManager.OpenQuestBoard
This is a public static coroutine that is exclusive to this interaction. It receives an EntityControl as the caretaker (the entity whose id is `data[0]`) and an NPCControl as the caller (which is this NPCControl).

Its job is to setup the opening of the quest board UI and also setups the [ItemList](../../../ItemList/ItemList.md) that shows the quests. Since this coroutine involves a lot of verbose rendering, its documentation is going to paraphrase a lot:
- All huds elements are hidden (including the berry count and discovery ones) by setting `discoveryhud`, `showmoney` and `hudcooldown` to 0.0
- The camera position is saved to the temporary files via SaveCameraPosition(true)
- [flag](../../../Flags%20arrays/flags.md) 2 (unchecked new quests) is set to true
- `option` is set to 0
- `minipause` is set to true
- The `No Quests` [BoardQuest](../../../Enums%20and%20IDs/BoardQuests.md) is removed from the `boardquests`
- A frame is yielded
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the caretaker to face the player
- Every `playerdata` entity has [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) called on them to face the caller
- instance.`boardcaller` is set to the caller
- instance.`camoffset` is set to the caller.`vectordata[0]`
- instance.`camangleoffset` is set to the caller.`vectordata[1]`
- instance.`camspeed` is set to the caller.`vectordata[2].x`
- caller.`vectordata[2].y` seconds are yielded
- Every `playerdata` entity has [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) called on them to face the caller
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the caretaker to face the player

From there, there is a lot of rendering logic that won't be detailed for brevity. The main things to keep in mind is the quest board root UI object is set to instance.`questboardobj` with the name `QuestBoard`. Every UI elements ends up being a descendant to it. This UI rendering involves [SetText](../../../SetText/SetText.md) calls in [non dialgue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode). The only thing worth mentioning about these calls is if the [languageid](../../../SetText/languageid.md) is `Japanese`, it affects the presence or parameters of [size](../../../SetText/Commands.md) commands and it doesn't offset the text of the tabs in x. If it's `Spanish`, the tab's text are offset by -0.75 in x. The tabs's text are offset by -0.05 in other languages.

From there, `listammount` is set to 10 and [ShowItemList](../../../ItemList/ShowItemList.md) is called with the [open quests list type](../../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md), a position of (-8.5, 2.55) with show description and without listsell.

All in all, this adds some functionality in MainManager.Update where the player is also able to use the left and right inputs to change the [listtype](../../../ItemList/listtype.md) being shown from open quests -> taken quests -> completed quests. This support wrap around. This functionality is possible thanks to the presence of the `questboardobj`.

Finally, UpdateQuestBoard is called and it will be called everytime the `listtype` changes.

## MainManager.UpdateQuestBoard
This is a private static method exclusive to this interaction. It is called by OpenQuestBoard and and also once everytime the player changes the board's [listtype](../../../ItemList/listtype.md).

All this really does is it manages the tabs's `sortingOrder` and whether or not the `Take Quest` UI element should be enabled (it only is on the [open quests list type](../../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)). The details on how this happens won't be documented for brevety.