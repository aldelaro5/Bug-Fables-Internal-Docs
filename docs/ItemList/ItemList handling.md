TODO: talk about MainManager Update


## ItemList

```cs
private static bool MultiItem()
```
Returns true if the current ItemList resulted in multiple items being selected in `multiselect`, false otherwise. More precisely, it returns true if `multiselect` isn't empty and at least one of the following condition is true:

- listtype is 2 (storage items)
- listtype is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

```cs
private static bool IsInMultiList()
```
Returns true if the current ItemList supports multiselect, false otherwise. More precisely, it returns true if at least one of the following condition is true:

- listtype is 2 (storage items)
- listtype is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

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

```cs
public void UpdateList(bool up)
public void UpdateList(bool up, int skip)
public void UpdateList(bool up, int skip, bool nosound)
public void UpdateList(Directions dir)
public void UpdateList(Directions dir, bool nosound)
```
Performs the necessary tasks and visual updates to reflect a change in the current `itemList` when the cursor moves. TODO: Document this in ItemList documentation if not done already
