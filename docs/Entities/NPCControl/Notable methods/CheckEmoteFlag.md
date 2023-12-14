# CheckEmoteFlag
This is a private method only used in [LateUpdate](../LateUpdate.md). It is called only under the following conditions:
- This isn't s `dummy`
- The entity is `incamera`
- Every 3 frames
- We aren't `inevent`
- We aren't in a `minipause`
- The entity isn't `dead` or `iskill` and `deathcoroutine` isn't in prgress
- The player `canpause`
- This NPCControl is an [NPC](../NPC)
- The player npc list is empty or the first element isn't this NPCControl

The method does nothing if `alwaysemoticon` is false and at least one of the following is true:
- The y distance between this NPCControl and the player is 2.0 or above
- The z distance between this NPCControl and the player is 15.0 or above

If we are doing something, we try to find the first `emoticonflag` that is applicable.

The way it works is NPCControl has a general purpose Vector2 array field called `emoticonflag` which contains for each element data an emoticon id and a [flag](../../../Flags%20arrays/flags.md) slot required to have it applied. The flag slot checks are performed in reverse order from the list, but one is always picked meaning the first one in the array serves as a falback. Here is how the components are defined (NOTE: despite all components being float, each gets cast to int for this which truncates the fractional part):

- x: A required [flag](../../../Flags%20arrays/flags.md) slot to be true in order for the line to be set as the current one. This is always violated if it's negative.
- y: The emoticon id related to this element

When the method finds the first element whose x's condition is satisfied in reverse order (or the first one if it got to it), it sets the entity `emoticonid` to the y component and `emoticoncooldown` to 5.0.
