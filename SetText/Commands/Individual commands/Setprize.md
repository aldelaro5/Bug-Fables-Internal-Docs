# Setprize

Mark a Caravan prize medal as obtained via its [Medal](../../../Enums%20and%20IDs/Medal.md) id or the caller's anim state which also removes it from the caravan list. This can also mark any prize medal as obtained via its prize id.

## Syntax

(1)

````
|setprize,find,medal|
````

(2)

````
|setprize,this|
````

(3) (Unused)

````
|setprize,prizeid|
````

(4) (Unused, same as (3))

````
|setprize,var,prizeid|
````

## Parameters

### `find`

This parameter indicates to operate in syntax (1) which marks the prize containing the `medal` as obtained and removed from the caravan list.

### `medal`: int | `v`int | `var`int

The [Medal](../../../Enums%20and%20IDs/Medal.md) id or a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing it to mark its corresponding prize as obtained and removed from the Caravan list. The int form must be a valid int or an exception will be thrown. The `v` and `var` form both means that the int part is a [flagvar](../../../Flags%20arrays/flagvar.md) slot and it must be a valid one otherwise, an exception will be thrown.

### `this`

This parameter indicates to operate in syntax (2) which marks the caller's [Medal](../../../Enums%20and%20IDs/Medal.md) as obtained and removed from the Caravan list.

### `prizeid`: int

The prize id to mark as obtained. This must be a valid prize id or an exception will be thrown. Since the prize arrays are hardcoded and there are 23 that exists in the game, a valid prize id is between 0 and 22.

### `var`

This parameter does nothing. Its value is completely ignored and specifying it is the same then omitting it.

## Remarks

This command's purpose is to mark a prize medal bough from the Caravan as obtained. This is not meant to be used with Artis's ones because these are handled in a different way using [Events](../../../Enums%20and%20IDs/Events.md). Syntax (1) is meant as a handler for using the [Caravan Prize Medals List Type](../../../ItemList/List%20Types%20Group%20Details/Caravan%20Prize%20Medals%20List%20Type.md) while syntax (2) is meant to handle buying the medal from the Caravan's counter because by the time this happens, the caller is the medal which knows its own [Medal](../../../Enums%20and%20IDs/Medal.md) id via its anim state.

If the prize medal doesn't exists, isn't available through the Caravan (the [flagvar](../../../Flags%20arrays/flagvar.md) slot of the prize medal is 2) or is already obtained, this command does nothing. If found, this command writes to the [flagvar](../../../Flags%20arrays/flagvar.md) slot bound to the prize medal (the value 3 which means obtained).

Syntax (3) and (4) are equivalent and unused in the game, but they allow to unconditionally mark any prize medal as obtained via their prize id.
