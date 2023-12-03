# Interaction
An `Interaction` is an enum that specify an interaction usually with the player (but it can be caused manually). The interaction of an NPCControl is given by the `interacttype` field. It only applies to [NPC](NPC.md) and [SemiNPC](Shop%20system.md#seminpc).

The actual execution of the interaction is done by [Interact](Notable%20methods/Interact.md) which is one of the only method in NPCControl intended to be called externally. It takes in one string parameter calld `args` which can change the logic of the interaction if applicable, but most do not make use of it. Some interactions have influences outside of this method however, some even outside NPCControl.

Most of the calls to that method comes from PlayerControl's GetInput when it detects the player attempts to interact and it will happen with the first NPCControl in the player.`npc` list if it is present under certain conditions. If PlayerControl calls it like that, null is sent as `args`. It is however possible to call it externally to provoke an interaction in the desired moment.

## Enum table
The following is a table of the different `Interaction` values (NOT REFERENCED means the game never reference the value meaning it has not logic backing it, UNUSED means the behavior has logic, but it is either dead or not practically used under normal gameplay):

|Value|Name|Description|
|----:|----|-----------|
|0|None|NOT REFERENCED, No operations|
|1|[Talk](Interaction/Talk.md)|Calls [SetText](../../SetText/SetText.md) in [Dialogue mode](../../SetText/Dialogue%20mode.md#dialogue-mode) using the text from the return of [GetDialogue](Notable%20methods/GetDialogue.md). It also gives the NPC a chatting emoticon when the NPC is in the `npc` interact list of the player.|
|2|[Check](Interaction/Check.md)|An alias of `Talk`, but the emoticon the NPC gets is the blue ? one when it is in the `npc` list of the player.|
|3|[SavePoint](Interaction/SavePoint.md)|The same than `Talk`, but the [SetText](../../SetText/SetText.md) input string is hardcoded. The string contains a [prompt](../../SetText/Individual%20commands/Prompt.md) command where the confirm option leads to a line with a [save](../../SetText/Individual%20commands/Save.md) command which will actually saves the game.|
|4|[Event](Interaction/Event.md)|Start the [event](../../Enums%20and%20IDs/Events.md) whose id is `eventid` with this NPC as the caller.|
|5|[TalkReturnToOriginalFlip](Interaction/TalkReturnToOriginalFlip.md)|UNUSED, An alias of `Talk`, but only the Interact logic remains.|
|6|[Shop](Interaction/Shop.md)|At loading time, this interaction is only for indicative purposes to build a [shop system](Shop%20system.md). At runtime after the shop system has been built, this is the interaction of a shelved item which is a `SemiNPC` [item entity](../EntityControl/Item%20entity.md) and the interaction calls [SetText](../../SetText/SetText.md) with the shop keeper's buying dialogue.|
|7|[ShopKeeper](Interaction/ShopKeeper.md)|Similar to `Talk`, but with a shop keeper buying or selling dialogue line.|
|8|[QuestBoard](Interaction/QuestBoard.md)|The interaction for a quest board which allows to consult the 3 quest boards and take any open quests by having a dialogue with the caretaker of the quest board which is another NPCControl.|
|9|[StorageAnt](Interaction/StorageAnt.md)|The same than `Talk`, but the [SetText](../../SetText/SetText.md) input string is hardcoded. It is the Ant Storage Service text unless [flag](../../Flags%20arrays/flags.md) 180 (received the Ant Storage Service tutorial) is false where it's the tutorial. This also sets [flag](../../Flags%20arrays/flags.md) 180 and 349 (using the Ant Storage Service, allows multiselect) to true.|
|10|[CaravanBadge](Interaction/CaravanBadge.md)|Similar than `Shop`, but for interacting with a shelved Caravan prize medal which only involve some parts of the [shop system](Shop%20system.md) because it is handled entirely on its own with its own set of data.|
|11|[VenusHeal](Interaction/VenusHeal.md)|The same than `Talk`, but the [SetText](../../SetText/SetText.md) input string is hardcoded to `commondialogue[60]` (the Venus bud healing text).|
|12|[LockedDoor](Interaction/LockedDoor.md)|Start [event](../../Enums%20and%20IDs/Events.md) 59 (Interacting with an object that requires a key item).|