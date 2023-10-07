# Addstat

Add a stat bonus by specifying its properties directly or by specifying [flagvar](../../Flags%20arrays/flagvar.md) slots containing them and optionally recalculate the party's stats after the removal.

## Syntax

(1)

````
|addstat,type,amount,applyto|
````

(2)

````
|addstat,type,amount,applyto,true|
````

## Parameters

### `type`: int | `var`int | `v`int | `money`

The stat bonus type to add to the stat bonuses list or the [flagvar](../../Flags%20arrays/flagvar.md) slot containing it. The int form must be a valid stat bonus type or an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. 

The `money` value is allowed, but is not recommended because it will read the current berry count as a stat bonus type which is likely to cause an exception to be thrown.

Here are the valid stat bonus type:

|Value|Stat increase|
|-----|-------------|
|0|HP|
|1|Attack|
|2|Defense|
|3|TP|
|4|MP|

### `amount`: int | `var`int | `v`int | `money`

The stat bonus amount or the [flagvar](../../Flags%20arrays/flagvar.md) slot containing it. The int form must be a valid int or an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. 

The `money` value is allowed, but is not recommended because it will read the current berry count as a stat bonus amount which is likely not desirable.

### `applyto`: int | `var`int | `v`int | `money`

The [AnimID](../../Enums%20and%20IDs/AnimIDs.md) of the recipient of the stat bonus or the [flagvar](../../Flags%20arrays/flagvar.md) slot containing it. The int form must be a valid [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) or -1 or an exception will be thrown. The [flagvar](../../Flags%20arrays/flagvar.md) string form must contain `var` or `v` only once with an integer portion corresponding to a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. 

-1 means this stat bonus will apply to the entire party.

The `money` value is allowed, but is not recommended because it will read the current berry count as the recipient which is likely not desirable.

### `true`

This parameter indicates to operate in syntax (2) which will also recalculate the party's stats after the stat bonus removal has been done. Any other value is ignored and will cause the command to operate in syntax (1).

## Remarks

For more information about how stat bonuses work and how they are saved on the [Save File](../../External%20data%20format/Save%20File.md), check [Save File > Line 10 (Array line) Stats bonuses](../../External%20data%20format/Save%20File.md#line-10-array-line-stats-bonuses).

Syntax (2) is recommended over syntax (1) because it allows the addition of the stat bonux to take immediate effect while syntax (1) will need to wait the next recalculation to take this into effect.

This command is only used for testing purposes in the TestRoom.
