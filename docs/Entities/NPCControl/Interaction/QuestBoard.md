# QuestBoard
Quest board interaction with a configurable caretaker map entity and flag slot ??? (TODO)

## args meaning
None.

## data meaning
- `data[0]`: A map entity id that corresponds to the caretaker of the quest board.
- `data[1]`: ???
- `data[2]`: If it's not negative, this is a [flag](../../../Flags%20arrays/flags.md) slot that if it's false, [Interact](../Interact.md) will be called on the `npcdata` of the map entity whose id is `data[0]`.

## LateUpdate (SpecialTattles)
The `tattleid` is set to -1.

## Interact
If `data[2]` isn't negative and its corresponding [flag](../../../Flags%20arrays/flags.md) slot is false, [Interact](../Interact.md) is called on the `npcdata` of the map entity whose id is `data[0]` without any arguments.

Otherwise, an OpenQuestBoard coroutine is started with the map entity whose id is `data[0]` as the caretaker and this as the caller.