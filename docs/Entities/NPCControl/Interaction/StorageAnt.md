# StorageAnt
The same than [Talk](Talk.md), but the [SetText](../../../SetText/SetText.md) input string is hardcoded. It is the Ant Storage Service text unless [flag](../../../Flags%20arrays/flags.md) 180 (received the Ant Storage Service tutorial) is false where it's the tutorial. This also sets [flag](../../../Flags%20arrays/flags.md) 180 and 349 (using the Ant Storage Service, allows multiselect) to true.

## args meaning
None.

## LateUpdate (Not a `dummy` and the entity is `incamera`)
If the [message](../../../SetText/Notable%20states.md#message) lock is released, we aren't `inevent` and it's not an [item entity](../../EntityControl/Item%20entity.md), the entity.[animstate](../../EntityControl/Animations/animstate.md) is set to entity.`basestate`.

## Interact
The same than [Talk](Talk.md), but with a few changes:

- [flag](../../../Flags%20arrays/flags.md) slot 349 is set to true (using the storage service, allows multiselect)
- The text used to call [SetText](../../../SetText/SetText.md) is `commondialogue[1]` (the Ant Storage Service text) unless [flag](../../../Flags%20arrays/flags.md) 180 (received the Ant Storage Service tutorial) is false. If it is false, the text is `|`[anim](../../../SetText/Individual%20commands/Anim.md)`,caller,Happy|` followed by `commondialogue[97]` (the Ant Storage Service tutorial).
- [flag](../../../Flags%20arrays/flags.md) 180 (received the Ant Storage Service tutorial) is set to true. This ensures the tutorial isn't given again if it was.

## MapControl.LateUpdate
There is no logic here unlike [Talk](Talk.md).

## PlayerControl.LateUpdate
The same than [Talk](Talk.md).
