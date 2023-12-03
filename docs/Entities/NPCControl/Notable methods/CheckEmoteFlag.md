#### CheckEmoteFlag
This is a private method only used in [LateUpdate](../LateUpdate.md).

This does nothing if `alwaysemoticon` is false, the y distance between this and the player is 2.0 or above and the z distance between this and the player is 15.0 or above.

If we are doing something, we try to find the first `emoticonflag` that is applicable.

The way it works is NPCControl has a general purpose Vector2 array field called `emoticonflag` which contains for each element data an emoticon id and a [flag](../../../Flags%20arrays/flags.md) slot required to have it applied. The flag slot checks are performed in reverse order from the list, but one is always picked meaning the first one in the array serves as a falback. Here is how the components are defined (NOTE: despite all components being float, each gets cast to int for this which truncates the fractional part):

- x: A required [flag](../../../Flags%20arrays/flags.md) slot to be true in order for the line to be set as the current one. This is always violated if it's negative.
- y: The emoticon id related to this element

When the method finds the first element whose x's condition is satisfied in reverse order (or the first one if it got to it), it sets the entity `emoticonid` to the y component and `emoticoncooldown` to 5.0.