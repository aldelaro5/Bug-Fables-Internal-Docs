# Removestat

Remove a stat bonus by its index or a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing it and optionally recalculate the party's stats after the removal.

## Syntax

(1)

````
|removestat,statbonus|
````

(2)

````
|removestat,statbonus,true|
````

## Parameters

### `statbonus`: int | `var`int | `v`int | `money`

The stat bonus index to remove from the stat bonuses list or the [flagvar](../../../Flags%20arrays/flagvar.md) slot containing it. The int form be a valid index of the stat bonuses list or an exception will be thrown. The [flagvar](../../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. The `money` value is allowed, but is not recommended because it will read the current berry count as a stat bonus index which is most likely not desirable.

### `true`

This parameter indicates to operate in syntax (2) which will also recalculate the party's stats after the stat bonus removal has been done. Any other value is ignored and will cause the command to operate in syntax (1).

## Remarks

For more information about how stat bonuses work and how they are saved on the [Save File](../../../Save%20File.md), check [Save File > Line 10 (Array line) Stats bonuses](../../../Save%20File.md#line-10-array-line-stats-bonuses).

Syntax (2) is recommended over syntax (1) because it allows the removal to take immediate effect while syntax (1) will need to wait the next recalculation to take the removal into effect.

This command is only used for testing purposes in the TestRoom.
