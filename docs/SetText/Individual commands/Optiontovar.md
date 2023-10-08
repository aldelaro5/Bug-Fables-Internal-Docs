# Optionvar

Set [flagvar](../../Flags%20arrays/flagvar.md) 0 or a specific slot to the last selected option from a [prompt](Prompt.md), [numberprompt](NumberPrompt.md) or [ItemList](../../ItemList/ItemList.md).

## Syntax

(1) (Set [flagvar](../../Flags%20arrays/flagvar.md) 0 to `option`)

````
|optionvar|
````

(2)

````
|optionvar,flagvar|
````

## Parameters

### `flagvar`: int

The [flagvar](../../Flags%20arrays/flagvar.md) slot to set the `option` value to. This must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

`option` is a notable field used to indicate the currently selected option starting from 0 in any [prompt](Prompt.md), [numberprompt](NumberPrompt.md) or [ItemList](../../ItemList/ItemList.md). It is an essential field for handling a confirmation with them. For more information, see [Prompt > Prompt handling](Prompt.md#prompt-handling), [NumberPrompt > About the fields, flagvar and flagstring used](NumberPrompt.md#about-the-fields-flagvar-and-flagstring-used) and [ItemList State Machine > Options management](../../ItemList/ItemList%20State%20Machine.md#options-management).
