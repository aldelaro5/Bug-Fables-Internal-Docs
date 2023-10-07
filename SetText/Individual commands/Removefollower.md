# Removefollower

Remove a follower from the `extrafollowers` list by [AnimID](../../Enums%20and%20IDs/AnimIDs.md) or remove all followers from it

## Syntax

(1) (Removes all followers by clearing the extrafollowers list)

````
|removefollower|
````

(2)

````
|removefollower,follower|
````

## Parameters

### `follower`: int

The follower's [AnimID](../../Enums%20and%20IDs/AnimIDs.md) to remove. This must be a valid [AnimID](../../Enums%20and%20IDs/AnimIDs.md) or an exception will be thrown.

## Remarks

For syntax (2), Only the first occurrence of the follower will be removed if multiple exists with the same [AnimID](../../Enums%20and%20IDs/AnimIDs.md). If no such follower exists, this command does nothing.

Since this only removes the follower(s) from the `extrafollowers` list, a map change needs to be done for the removal to take effect because the follower(s) is still in the `map.tempfollowers` list if it was allowed by the `canfollowID` whitelist.
