# Checkregional

Redirect to another dialogue line using a [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) if a [Regionalflag](../../Flags%20arrays/Regionalflags.md) slot is false or if it is in a specific state.

## Syntax

(1)

````
|checkregional,regional,lineidredirect|
````

(2)

````
|checkregional,regional,lineidredirect,statecheck|
````

## Parameters

### `regional`: int

The [Regionalflag](../../Flags%20arrays/Regionalflags.md) slot to check the state. This must be a valid [Regionalflag](../../Flags%20arrays/Regionalflags.md) slot or an exception will be thrown.

### `lineidredirect`: int

The [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect if the regional slot is false or if its state is `statecheck`. This must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

### `statecheck`: `true` | `false`

The state to compare the regional slot against. This must be `true` or `false` or an exception will be thrown.

## Remarks

The redirect is done using an [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue line resolved using `lineidredirect` prepended with |[blank](Blank.md)\|. This also ends the [Text advance](../Related%20Systems/Text%20advance.md)'s `skiptext`.

If a redirection happens, processing will resume at the start of the new input string.
