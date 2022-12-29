# Addprize

Mark a Caravan prize medal as available via its prize id.

## Syntax

````
|addprize,prizeid|
````

## Parameters

### `prizeid`: int

The prize id to mark as available. This must be a valid prize id or an exception will be thrown. Since the prize arrays are hardcoded and there are 23 that exists in the game, a valid prize id is between 0 and 22.

## Remarks

This also increments [flagvar](../../../Flags%20arrays/flagvar.md) 55 (number of prize medals available or obtained).

The actual availability depends on whether the Hard Mode [Medal](../../../Enums%20and%20IDs/Medal.md) is on or if [flags](../../../Flags%20arrays/flags.md) 614 (HARDEST) is on. If either of them are, the [flagvar](../../../Flags%20arrays/flagvar.md) corresponding to the prize is set to 1 (available through Artis). If neither are on, it will be set to 2 instead (available for purchase at the Caravan).
