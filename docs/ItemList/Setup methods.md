# Setup methods
Before calling [ShowItemList](ShowItemList.md), several ItemList related fields needs to be set, but there's 2 methods that does all of that: SetUpList and ResetList.

## SetUpList

```cs
public static void SetUpList(int type, bool showdescription, bool sell)
```
This is a helper to preconfigure an ItemList in a standard way before calling ShowItemList. It shouldn't be called during an ItemList processing, only before starting one. It sets some ItemList related fields:

- `storeid`: 0
- `listtype`: `type`
- `listammount`: 6
- `listdesc`: `showdescription`
- `listsell`: `sell`

The method also sets `inputcooldown` to 5.0 so most inputs processing are disabled for 5.0 frames.

## ResetList

```cs
public static void ResetList()
```
Sets some ItemList related fields to their starting values. This is meant to be a helper to ShowItemList where calling this can help prepare the setup of the ItemList before calling ShowItemList. It shouldn't be called during an ItemList processing, only before starting one. Here are the fields and their values:

- `listcursor`: 0
- instance.`option`: 0
- `listlow`: 0
- `listmax`: `listammount`
- `listY`: -1
