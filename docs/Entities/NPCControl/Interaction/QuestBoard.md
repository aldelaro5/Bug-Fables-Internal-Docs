# QuestBoard
Quest board interaction with a configurable caretaker map entity and flag slot ??? (TODO)

## args meaning
None.

## Data meaning
- `data[0]`: A map entity id that corresponds to the caretaker of the quest board.
- `data[1]`: ???
- `data[2]`: If it's not negative, this is a [flag](../../../Flags%20arrays/flags.md) slot that if it's false, [Interact](../Interact.md) will be called on the `npcdata` of the map entity whose id is `data[0]`.
- `vectordata[0]`: ???
- `vectordata[1]`: ???
- `vectordata[2].x`: ???
- `vectordata[2].y`: ???

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
This is a public static coroutine that is exclusive to this interaction.

TODO

## MainManager.UpdateQuestBoard
This is a private static method exclusive to this interaction. It is called by OpenQuestBoard and and also once everytime the player changes the board's [listtype](../../../ItemList/listtype.md).

TODO