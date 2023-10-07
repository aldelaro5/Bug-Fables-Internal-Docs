# PickItem

Setup an [ItemList](../../ItemList/ItemList.md) using [ShowItemList](../../ItemList/ShowItemList.md) with the parameters sent in [Dialogue mode](../Dialogue%20mode.md) which can get handled by SetText after the ItemList goes inactive.

## Syntax

(1)

````
|pickitem,type,redirect,cancel|
````

(2)

````
|pickitem,type,storeid,desc,sell,redirect,cancel|
````

(3)

````
|pickitem,type,storeid,desc,sell,redirect,cancel,ammount|
````

## Parameters

### `type`: int

The [listtype](../../ItemList/listtype.md). This must be a valid int value or an exception will be thrown. For more information on how each value behaves, consult the [listtype](../../ItemList/listtype.md) document.

### `redirect`: int

The `listredirect`. This must be a valid int value or an exception will be thrown. This is the desired [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect when the [ItemList](../../ItemList/ItemList.md) is confirmed. There are special values that can be specified which are exceptions to this:

* null: No redirection, just run the post ItemList handler.
* -1: This acts like null.
* -2: No redirection, but assume in the post ItemList handler that this an [items](../../Enums%20and%20IDs/Items.md) was selected to be tossed after a [additemtoss](Additemtoss.md) command. By this point, the selected item id must be in [flagvar](../../Flags%20arrays/flagvar.md) 1 and the original item passed in [additemtoss](Additemtoss.md) to be in [flagvar](../../Flags%20arrays/flagvar.md) 0. The caller (which is assumed to be the item above the player's [Entity](../../Entities/Entity.md)) will have its appearance set to the selected item. It will get tossed in a random direction and disappear after 600 frames. This will also remove the selected item id from the inventory of the `itemtype` sent to [additemtoss](Additemtoss.md) (which is now the [listtype](../../ItemList/listtype.md)) and it will add the item id sent to [additemtoss](Additemtoss.md) in that same inventory.
* -3: No redirection, same as -2, but assume that we are tossing the same item sent to [additemtoss](Additemtoss.md) which means everything happens the same way except the inventory operations and the change of the caller's appearance.

-2 and -3 should never be sent outside of an [additemtoss](Additemtoss.md) context or undefined behaviors will occur.

### `cancel`: int

The `listcancel`. This must be a valid int value or an exception will be thrown. This works the same way than `redirect`, but the value will only become the outcome in the post [ItemList](../../ItemList/ItemList.md) handler if the list has been cancelled.

### `storeid`: int

The `storeid`. This must be a valid int value or an exception will be thrown. The default value is 0.

### `desc`: `true` | `false`

The `listdesc`. This must be equal to `true` or `false` without case sensitivity or an exception will be thrown. The default value is `true`.

### `sell`: `true` | `false`

The `listsell`. This must be equal to `true` or `false` without case sensitivity or an exception will be thrown. The default value is `false`.

### `amount`: int

The `listammount`. This must be a valid int value or an exception will be thrown. The default value is 5.

## Remarks

This command will first reset `multiselect` to a new list and an `inputcooldown` of 5 frames is applied. It will then set all the [ItemList State Machine](../../ItemList/ItemList%20State%20Machine.md) fields mentioned above to each of their respective parameter value.

After, it will call [ShowItemList](../../ItemList/ShowItemList.md) with `type` as the type, (4.0, 0.5) as the position, `listdesc` as the `showdescription` and `listsell` as the sell.

This command requires [Dialogue mode](../Dialogue%20mode.md) otherwise, SetText will not wait for the [ItemList](../../ItemList/ItemList.md) to finish processing and as such, it will not get handled.

## Post ItemList handler

After the call is made, the regular [ItemList](../../ItemList/ItemList.md) system kicks in just like it would without SetText. For more information on how this system and its state machine works, consult the [ItemList](../../ItemList/ItemList.md) document. The key difference is SetText will handle the outcome of the [ItemList](../../ItemList/ItemList.md) which is `listredirect` in the case of a confirmation and `listcancel` in the case of a cancellation. This is done in 2 parts during the [Dialogue post-processing#ItemList handling](../Life%20Cycle.md#dialogue-post-processing-itemlist-handling)

Additionally, if -2 or -3 was the outcome, [end](End.md) is set to true, `listredirect` is set to null, an `actioncooldown` of 30 frames is applied and `eventcall` is set to the one determined from the [additemtoss](Additemtoss.md) command processing.
