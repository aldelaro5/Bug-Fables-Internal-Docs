# UpdateItem
This is an update method that only applies to [item entities](../Item%20entity.md). It only does anything if the `sprite` exists.

- The `sprite` is disabled if the `animid` is 3 (this is a Crystal Berry)
- If the `animid` is 0 or 1 (this is a standard or key item), the `sprite` is assigned to the corresponding `itemsprite[0, x]` where x is the `itemstate` which is the [item](../../../Enums%20and%20IDs/Items.md) id.
- If the `animid` is 2 (this is a medal), the `sprite` is assigned to the corresponding `itemsprite[1, x]` where x is the `itemstate` which is the [medal](../../../Enums%20and%20IDs/Items.md) id. There is an exception to this: if [flag](../../../Flags%20arrays/flags.md) slot 681 is true (MYSTERY? is active), `overridemovesmake` is false and `itemstate` isn't 59 (Extra Freeze), the `sprite` is assigned to `guisprites[190]` instead (the ? medal sprite)

After everything, if the `sprite` is enabled with a backing sprite, the local position of `spritetransform` is set to (0.0, the y extend of the `sprite`'s sprite, 0.0).