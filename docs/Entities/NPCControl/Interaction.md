# Interaction
An `Interaction` is an enum that specify an interaction usually with the player (but it can be caused manually). The interaction of an NPCControl is given by the `interacttype` field. It only applies to [NPC](NPC.md) and [SemiNPC](Shop%20system.md#seminpc).

The actual execution of the interaction is done by [Interact](Notable%20methods/Interact.md) which is one of the only method in NPCControl intended to be called externally. It takes in one string parameter calld `args` which can change the logic of the interaction if applicable, but most do not make use of it. Some interactions have influences outside of this method however, some even outside NPCControl.

Most of the calls to that method comes from PlayerControl's GetInput when it detects the player attempts to interact and it will happen with the first NPCControl in the player.`npc` list if it is present under certain conditions. If PlayerControl calls it like that, null is sent as `args`. It is however possible to call it externally to provoke an interaction in the desired moment.

## Enum table
The following is a table of the different `Interaction` values (NOT REFERENCED means the game never references the value meaning it has no logic):

|Value|Name|Description|
|----:|----|-----------|
|0|None|NOT REFERENCED, No operations|
|1|[Talk](Interaction/Talk.md)| |
|2|[Check](Interaction/Check.md)| |
|3|[SavePoint](Interaction/SavePoint.md)| |
|4|[Event](Interaction/Event.md)| |
|5|[TalkReturnToOriginalFlip](Interaction/TalkReturnToOriginalFlip.md)| |
|6|[Shop](Interaction/Shop.md)| |
|7|[ShopKeeper](Interaction/ShopKeeper.md)| |
|8|[QuestBoard](Interaction/QuestBoard.md)| |
|9|[StorageAnt](Interaction/StorageAnt.md)| |
|10|[CaravanBadge](Interaction/CaravanBadge.md)| |
|11|[VenusHeal](Interaction/VenusHeal.md)| |
|12|[LockedDoor](Interaction/LockedDoor.md)| |