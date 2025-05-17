
## Followers

```cs
public static bool HasFollower(AnimIDs followerid)
```
Returns true if any `tempfollowers` has an `originalid` of `followerid` - 1, false otherwise.

```cs
public static void TeleportFollowers()
public static void TeleportFollowers(bool all)
```
Does the following to bring all the followers close to the `player`:

- All `playerdata` except the first one has their `entity`.`rigid`.velocity reset to Vector3.zero and their `entity` poisition set to the `player` position + `globalcamdir`.forward.normalized * 0.025
- If `all` is true, all `map`.`tempfollowers` has their position set to the `player` position
- If `map`.`chompy` is present, DelayedPosition is called on them with a pos of the `player` position + `globalcamdir`.forward.normalized * 0.3

The parameterless overload has the `all` value default to false.

```cs
public static void TeleportFollowers(float distance, TPDir dir, Transform caller)
public static void TeleportFollowers(float distance, TPDir dir, Vector3 caller)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller, bool alsoTPextras)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller)
public static void TeleportFollowers(TPDir direction, Vector3 from)
```
TODO: This is complex, document and organise later

```cs
private static EntityControl GetLastFollowing()
```
Returns the last entity in the `player` follow chain or `player`.`entity` if the `player`.`entity` has no `followedby`. This will prioritise the last `playerdata`'s `entity`, but if that one has a `followedby`, the last `map`.`tempfollowers` is returned instead.

```cs
public static EntityControl AddFollower(EntityControl caller, int id)
```
Adds a follower to the follow chain. Check the follower system documentation to learn more.

```cs
public static EntityControl GetExtraFollower(int id)
```
Returns the first follower entity in `map`.`tempfollowers` whose animid is `id` or null if no such follower exists.