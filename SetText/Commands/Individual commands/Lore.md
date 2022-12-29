# Lore

A helper of [String](String.md) where [flagstring](../../../Flags%20arrays/flagstring.md) 0 is set to the Lore Book's text whose id is in [flagvar](../../../Flags%20arrays/flagvar.md) 0 with additional commands then the same [flagstring](../../../Flags%20arrays/flagstring.md) is sent to [String](String.md) in syntax (4).

## Syntax

````
|lore|
````

## Parameters

None.

## Remarks

More specifically, this command sets [flagstring](../../../Flags%20arrays/flagstring.md) 0 to the Lore Book text appended with |[break](Break.md)\||[hide](Hide.md)\||[fwait](Fwait.md),0.05||[boxstyle](Boxstyle.md),3||[fwait](Fwait.md),0.05||[hide](Hide.md)\||[goto](Goto.md),1|. Then, a |[string](String.md),0,true| is immediately processed which will use the newly set value of [flagstring](../../../Flags%20arrays/flagstring.md) 0.

This is meant to be used as a confirmation handler to [Lore Book List Type](../../../ItemList/List%20Types%20Group%20Details/Lore%20Book%20List%20Type.md) which would have already set [flagvar](../../../Flags%20arrays/flagvar.md) 0 to the Lore Book id. This command simply does the setup needed to render it.
