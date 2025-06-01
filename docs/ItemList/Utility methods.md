# ItemList utility methods
Here are various methods to work with ItemLists not documented elsewhere. They belong in MainManager.

## `multiselect` Itemlists
Some ItemLists supports multi items selection. These methods deals with that feature.

```cs
private static bool MultiItem()
```
Returns true if the current ItemList resulted in multiple items being selected in `multiselect`, false otherwise. More precisely, it returns true if `multiselect` isn't empty and at least one of the following condition is true:

- [listtype](listtype.md) is 2 (storage items)
- [listtype](listtype.md) is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

```cs
private static bool IsInMultiList()
```
Returns true if the current ItemList supports multiselect, false otherwise. More precisely, it returns true if at least one of the following condition is true:

- [listtype](listtype.md) is 2 (storage items)
- [listtype](listtype.md) is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

## DestroyList
This method is useful to smoothly stop the rendering of the current ItemList when it is done.

```cs
private void DestroyList()
```
Destroys the current ItemList to stop its rendering with a shrink animation done over 0.5 seconds.

This coroutine also does the following additionally:

- Set `itemList` to null
- Destroys `cursor`
- Reset `skiptext` to false
- `listoption` is reset to `option`
- `option` is reset to -1
- `listY` is reset to -1

NOTE: Calling this doesn't necessarily mean the ItemList is done being processed. It only means that the UI will no longer show, but it's possible the game needs to process the outcome of the ItemList after. This happens for example with SetText and a PickItem command where SetText needs to process the outcome after `itemList` has been set to null. This is an important distinction for the `inlist` issue.

## UpdateList
Most the the handling of the naviguation of an ItemList is done through [MainManager.Update](../MainManager/Update%20and%20FixedUpdate.md), but it also uses a method that specifically handles some ItemList updates: UpdateList.

```cs
public void UpdateList(Directions dir)
public void UpdateList(Directions dir, bool nosound)
```
Performs the necessary tasks and visual updates to reflect a change in the current `itemList` when the cursor moves in the direction `dir` (only `Up` and `Down` are supported) with a scroll sound unless `nosound` is true. It's meant to be called by whoever is managing the ItemList's naviguation which is either MainManager.Update or PauseMenu.Update. This method doesn't support wrap around.

The overload without a `nosound` parameter has the value default to false.

Here's what happens when `dir` is `Down`:

- If `option` + 1 is `maxoptions` or higher, nothing happens and the method returns early (because it means the selected item is the bottommost in the ItemList)
- If `nosound` is false, PlayScrollSound is called
- `option` gets incremented
- If `listmax` + 1 is at most `maxoptions` (which normally is always the case) and `option` is the same as `listmax` (meaning it is at the bottommost option of the visible segment of the ItemList):
    - Both `listlow` and `listmax` gets incremented which shifts the visible segment of the ItemList by 1 down
- Otherwise (the `option` is still within the visible segment of the ItemList):
    - `listcursor` gets incremented which changes which item is selected in the ItemList, but doesn't change the visible segment range

Here's what happens if `dir` is `Up`:

- If `option` - 1 is less than 0, nothing happens and the method returns early (because it means the selected item is the topmost in the ItemList)
- If `nosound` is false, PlayScrollSound is called
- `option` gets decremented
- If `listlow` - 1 is at most 0 (which normally is always the case) and `option` is the same as `listlow` - 1 (meaning it is past the topmost option of the visible segment of the ItemList):
    - Both `listlow` and `listmax` gets decremented which shifts the visible segment of the ItemList by 1 up
- Otherwise (the `option` is still within the visible segment of the ItemList):
    - `listcursor` gets decremented which changes which item is selected in the ItemList, but doesn't change the visible segment range

There's also 3 other overloads that calls the above, but extends the logic a bit to allow multiple consecutive calls with wrap around:

```cs
public void UpdateList(bool up)
public void UpdateList(bool up, int skip)
public void UpdateList(bool up, int skip, bool nosound)
```
Here, `up` gets converted to a dir value (true is `Up`, false is `Down`) and it will call the above overload `skip` amount of times using the converted dir value and the `nosound` value as is.

This method also adds wrap around support: if `up` is true while `option` is 0 (the first option), the UpdateList overload above is called with a dir of `Down` continuously until it reaches the last option and sets `listY` to -1 (which will force a full refresh on the next [ShowItemList](ShowItemList.md) call). Likewise, if `up` is false while `option` is `maxoption` - 1 (the last option), the UpdateList overload above is called with a dir of `Up` continuously until it reaches the first option and sets `listY` to -1 (which will force a full refresh on the next [ShowItemList](ShowItemList.md) call).

All overloads without a `nosound` parameter has the value default to false.

All overloads without a `skip` paramter has the value default to -1.
