# AddItemToss

Give an [Items](../../../Enums%20and%20IDs/Items.md) or [medal](../../../Enums%20and%20IDs/Medal.md) based on its id or a [flagvar](../../../Flags%20arrays/flagvar.md) containing it. If it's an item and the item inventory is full, prompt to toss an item.

## Syntax

(1)

````
|additemtoss,itemtype,itemid|
````

(2)

````
|additemtoss,itemtype,var,flagvar|
````

## Parameters

### `itemtype`: `0` | `1` | `2`

The inventory type this command should attempt to give the item:

* 0: Standard items.
* 1: Key items. 
* 2: Medals (it will be added as unequipped).
* 3: This is ignored and no items will be added.

Any other value will cause an exception to be thrown.

### `var`

This is just what determines how the item id is obtained. Any other value will this parameter to be interpreted as the `itemid`.

### `itemid`: int

The item or medal id to attempt to give. This must be a valid int value or an exception will be thrown.

### `flagvar`:  int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the item id. This must corresponds to an int of a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

## Remarks

The toss prompt only happens if `itemtype` is 0 and the current standard item inventory is full. If we are into this situation, the caller will get some adjustments before displaying the prompt:

* Its hit is set to false.
* Its descwindow gets destroyed.
* eventtoss is set to its `data[1]` if it exists and is above -1 

To display this prompt, the input string will be redirected to the toss prompt line from menu text 3 prepended with |[blank](Blank.md)\||[boxstyle](Boxstyle.md),4||[spd](Spd.md),0| and appended with |[pickitem](Pickitem.md),0,1,true,false,-2,-3,5|. The blinker is then disabled and processing will resume at the start of the new input string. This [Pickitem](Pickitem.md) will be handled differently than usual at [life cycle > Additemtoss Special sub-handler](../../life%20cycle.md#additemtoss-special-sub-handler)

In the other case where the item can be given, it will be given immediately (if `itemtype` is 3, nothing will happen here). After, the caller's descwindow gets destroyed and finally, an [End](End.md) command is relayed.
