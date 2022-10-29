# [ShowItemList](ShowItemList.md) Life Cycle

See [ItemList State Machine](ItemList%20State%20Machine.md) to learn about the fields mentioned.

## Setup

* Sets `listy` to -1 and `multiselect` to a new list if we are creating a new ItemList.
* Sets `listpos` to `position`.
* Sets `listcanceled` to false.

## `listvar` generation

Check the individual [listtype](listtype.md) to learn more.

## Rerendering after scroll

* if `listlow` isn't `listy` (meaning the list needs to be scrolled or it's a new ItemList) or [listtype](listtype.md) is the language selection list: 
  * [Rerendering after scroll](ShowItemList%20Life%20Cycle/Rerendering%20after%20scroll.md)

## `listdescbox` rendering

* if `showdescription` and `listvar` isn't empty.
  * Destroys the existing `listdescbox` if it isn't null.
  * [Description box rendering](ShowItemList%20Life%20Cycle/Description%20box%20rendering.md)
* Sets the `ItemList`'s localEulerAngle to Vector3.zero.
