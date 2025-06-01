# GetDialogueFromMap
There is a method that allows to obtain any map SetText dialogue line from any map even if it's not the current map. This method is GetDialogueFromMap:

```cs
public static string GetDialogueFromMap(Maps mapid, int lineid, int boxid)
```
It gets the entire SetText line whose id is `lineid` from the map `mapid` if `boxid` is -1.

If `boxid` isn't -1, this will get a specific parts of the SetText line using the `boxid` as the index in an array produced by splitting the SetText line by `|next|` (with StringSplitOptions.None meaning empty strings are included in the resulting split array).

This is used as part of the [getfrommap](../Individual%20commands/GetFromMap.md) SetText command and some specific events.

NOTE: If `boxid` isn't a valid index of the string after it is split by `|next|`, an exception will be thrown. Also, this doesn't support the typical dialogue line id format, only a map line id is supported and the id must be valid inside the `mapid` map or an exception will be thrown.
