# GetDialogue
This method not only returns the [SetText](../../../SetText/SetText.md) line the [NPC](../NPC.md) has at the current moment, it also configures some fields that this line needs.

## Normal process
The way it works is NPCControl has a general purpose Vector3 field called `dialogues` which contains for each element data about a SetText line assigned to the entity. The order of the elements is important: the method will search for the first suitable one to set as current in reverse order (from last to first).

Here is how the components of an element are defined (NOTE: despite all components being float, each gets cast to int for this which truncates the fractional part):

- x: A required [flag](../../../Flags%20arrays/flags.md) slot to be true in order for the line to be set as the current one. If it's <= -1, it means no flag is required and the line will be set as the current one when encountered. If the condition is violated, the method will ignore this element and check the next one.
- y: The [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) related to this element. Because it uses the same scheme SetText commands uses, it supports both map specific line id or common dialogue line id.
- z: The `basestate` and `animstate` to set the entity when this line is set as the current one. The `basestate` is the default [animstate](../../EntityControl/Animations/animstate.md) the entity has instead of the default which is `Idle` or 0. This allows to force the entity into a different animation when the line is selected as current.

When the method finds the first element whose x's condition is satisfied in reverse order, it sets `currentdialogueindex` to the element's index, the entity's `basestate` to the z component and it returns the actual SetText line corresponding to the y component.

## Override dialogue feature
There is however an exception to this: it's possible to override this logic by setting `overridediag` to a non negative value which will be used as the [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) instead. This feature is never used under normal gameplay and this field's default value remains -1. It should be noted that this feature specifically supports the incomplete [global commands system](../../../SetText/Related%20Systems/GlobalCommand.md) and it supports [SavePoint](../Interaction/SavePoint.md) interactions unlike the normal process.

## Error handling and exceptional cases
If the `interacttype` is [Shop](../Interaction/Shop.md) or [SavePoint](../ObjectTypes/SavePoint.md) or none of the elements in `dialogues` field is suitable to set as current, no fields are touched and the return value is hardcoded to `|`[color](../../../SetText/Individual%20commands/Color.md)`,1|Invalid message.`. This should never happen under normal gameplay because this is supposed to be a debug message that isn't supposed to be seen if every entities are correctly configured. This is only for diagnosis purposes.