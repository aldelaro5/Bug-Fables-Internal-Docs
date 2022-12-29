# Checksum

Process a [Goto](Goto.md) in syntax (1) if 2 values or [flagvar](../../../Flags%20arrays/flagvar.md) slots added together is strictly bigger than another value or [flagvar](../../../Flags%20arrays/flagvar.md) slot or do nothing otherwise.

## Syntax

````
|checksum,value1,value2,checkvalue,redirect|
````

## Parameters

### `value1`: int | `v`int | `var`int | `money`

The first value or [flagvar](../../../Flags%20arrays/flagvar.md) slot to add together. The int form must be a valid int or an exception will be thrown. The `v`/`var` prefix indicates to use the value of a [flagvar](../../../Flags%20arrays/flagvar.md) slot and it must be a valid one or an exception will be thrown. `money` indicates to use the current berries count as the value.

### `value2`: int | `v`int | `var`int | `money`

The second value or [flagvar](../../../Flags%20arrays/flagvar.md) slot to add together. The int form must be a valid int or an exception will be thrown. The `v`/`var` prefix indicates to use the value of a [flagvar](../../../Flags%20arrays/flagvar.md) slot and it must be a valid one or an exception will be thrown. `money` indicates to use the current berries count as the value.

### `valuecheck`: int | `v`int | `var`int | `money`

The value or [flagvar](../../../Flags%20arrays/flagvar.md) slot to test the sum of `value1` and `value2`. The int form must be a valid int or an exception will be thrown. The `v`/`var` prefix indicates to use the value of a [flagvar](../../../Flags%20arrays/flagvar.md) slot and it must be a valid one or an exception will be thrown. `money` indicates to use the current berries count as the value. The redirection only happens if `value1` + `value2` is strictly bigger than this value.

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to send to [Goto](Goto.md) if applicable. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown if [Goto](Goto.md) is processed with it.

## Remarks

The [Goto](Goto.md) command is immediately processed if applicable.
