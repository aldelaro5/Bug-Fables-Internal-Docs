# Followers
While `playerdata[0].entity` has the PlayerControl and the party is setup with a follow system so they follow each other when the `player` moves, the player party aren't the only ones allowed to follow the `player`.

As explained in the EntityControl's [Follow](../Entities/EntityControl/Notable%20methods/Follow.md) documentation, there's a follow chain system where an entity's `follow` tells if the game will want them to follow another entity. This followee could in turn have a `follow` so it can form a chain which stops when an entity's `follow` is null. Likewise, it's possible to traverse the chain backwards using the entity's `followedby`.

This implies that an entity could follow the last `playerdata` and that entity could in turn get followed, etc... What's interesting about this is that any other entity than the player party is managed into its own system called the [follower system](../MapControl/Follower%20system.md). They are stored in the `map`.`tempfollowers` with `chompy` being managed separately.

This is what this page is aimed to document. While it is mostly configured and controlled by MapControl which uses it in a specific way (such as the special `chompy` follower), the underlying system is globally accessible and it's frequently accessed for reasons such as quest events or requests from SetText. Because of this, the high level functionality is documented here, but the configuration and MapControl specific parts will be documented separated in the map followers part of the MapControl documentation.

## AddFollower
To add a follower at the end of the follow cahin, a call to AddFollower is needed:

```cs
public static EntityControl AddFollower(EntityControl caller, int id)
```
There are 2 ways to use this method:

- Adding an existing entity as follower: Doing this involves recreating the entity by first destroying it before creating a new one at the same position and the new one will become the follower. To do this, `caller` should be not null and correspond to the entity to add as follower in this fashion. In this mode of operation, `id`'s value won't be used and it will be overriden to the `caller`'s animid
- Adding a follower from a completely new entity: Doing this simply involves creating the entity and adding it as follower. To do this, `caller` needs to be null and `id` needs to contain a valid [animid](../Enums%20and%20IDs/AnimIDs.md)

Here's the procedure this method does to add the follower and store it into the `map`.`tempfollowers`:

- If caller isn't null (meaning this entity needs to be destroyed before recreating it):
    - caller is [unfixed](../Entities/EntityControl/EntityControl%20Methods.md#unfix)
    - The id parameter is overriden to the caller's `animid` and it is added to instance.`extrafollowers` (these are followers stored on the save file)
    - The caller is destroyed
- If id is 0 or above:
    - The last entity in the follow chain is obtained. It's the `player` if it's not `followedby` anyone otherwise it's the last `playerdata`.entity if that one isn't `followedby` anyone otherwise it's the last `map`.`tempfollowers`
    - [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) is called to create an entity with id as the animid, `Follower X` as the name where `X` is the id and the follow being the last entity in the follow chain obtained earlier. The position depends on if this is replacing an entity or not:
        - If it's replacing an entity, it's the position of that entity before it was destroyed
        - If it's not replacing an entity, the position is the `player` position + instance.`globalcamdir`.forward.normalized * the length of `playerdata` * 0.05 (this prevents z fighting by offsetting the follower slightly towards the camera)
    - If `map` exists (it's not being loaded), the newly created entity is childed to the `map` (otherwise, the entity remains rooted)
    - The entity gets a `PFollower` tag and its `tempfollower` field is set to true which marks it as a player follower. This registers them to be implicated in the EntityControl [follow system](../Entities/EntityControl/Notable%20methods/Follow.md)
    - The `player`'s `npc` list is reset to a new list
    - The newly created entity is added to the `map`.`tempfollowers`
    - The last entity in the follow chain has its `followedby` set to the newly created entity which completes the link of the new follower in the chain

As for the return value, it's not used by the game, but the method returns the entity whose `followedby` was set to the new follower. It's possible the method returns null, but it's only under an erroneous condition and that is that the value of id (after it's overriden if applicable) is negative which means it's not a valid animid. This isn't supposed to happen because normally, either `caller` is specified and its `animid` isn't negative or `caller` is null, but id has a non negative value. It's still possible that neither case are true and in which case, null will be returned, but it indicates that an invalid follower was attempted to be added.

MapControl has a specific way to use this method on each map load. Check the map [followers](../MapControl/Follower%20system.md) documentation to learn more. NOTE: There are 2 known issues with the way AddFollower works with MapControl, but they are documented in the MapControl part of the followers documentation.

## TeleportFollowers
For various reasons, the game often wants to visually move the player party and their followers in a certain organised fashion. This functionality is done through various overloads with the same name: TeleportFollowers.

There's a simple and a more complex version. Here is the simple version:

```cs
public static void TeleportFollowers()
public static void TeleportFollowers(bool all)
```
It does the following to bring all the followers close to the `player`:

- All `playerdata` except the first one has their `entity`.`rigid`.velocity reset to Vector3.zero and their `entity` poisition set to the `player` position + `globalcamdir`.forward.normalized * 0.025
- If `all` is true, all `map`.`tempfollowers` has their position set to the `player` position
- If `map`.`chompy` is present, DelayedPosition is called on them with a pos of the `player` position + `globalcamdir`.forward.normalized * 0.3

The parameterless overload has the `all` value default to false.

Here is the complex version:

```cs
public static void TeleportFollowers(TPDir direction, Vector3 from)
public static void TeleportFollowers(float distance, TPDir dir, Transform caller)
public static void TeleportFollowers(float distance, TPDir dir, Vector3 caller)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller, bool alsoTPextras)
```
It moves all `playerdata` (except the first one) whose `entity` have a `following` entity and their GetDistance between themselves and their `following` entity is higher than `distance`. The movement is done relative to `caller` using `dir` as the direction to use with a distance of `offset` between each `playerdata`. Alongside the movement, their `entity.rigid` also has their velocity reset to Vector3.zero.

More precisely, here's what each value of `dir` represents (these are all normalized or very small as the new `playerdata` overworld entity position will be that direction * `offset` + their `following` entity position):

- `Right`: `globalcamdir`.right.normalized
- `Left`: -`globalcamdir`.right.normalized
- `Up`: `globalcamdir`.forward.normalized
- `Down`: -`globalcamdir`.forward.normalized
- `Away`: -(`caller` - the `following` entity position).normalized + `MainCamera`.transform.forward.normalized * 0.1. Intuitively, this is the direction from the `caller` to the `following` entity + the camera's forward direction scaled to a magnitude of 0.1 (this helps with z fighting)
- `Center`: `globalcamdir`.forward.normalized * 0.1 * (the `playerdata` index + 1). This essentially moves the entities the least amount as possible, but still has z fighting prevention among the `playerdata`

Additionally, if `map`.`chompy` esists, has a `following` entity and the GetDistance between it and `map`.`chompy` is more than `distance`, they are moved to their `following` entity position + `globalcamdir`.forward.noramlized * 0.1 (this prevents z fighting).

`alsoTPExtras` should always be false because if it's true, it will cause the `map.tempfollowers` to be moved too, but it uses broken logic to do so as they would have always been moved from the `caller` instead of their `following` entity and the vector math is in general illogical. It also prevents the `map`.`chompy` entity from being moved. The game never sends true to this parameter under normal gameplay and all overloads without a `alsoTPExtras` parameter has its value default to false.

For the first 3 overloads, the `offset` parameter defaults to 1.0.

For the second overload, `caller`.position is used for the `caller` Vector3 value.

For the first overload, `distance` defaults to 1.25 and the `caller` parameter is mapped from the `from` parameter it receives.

## General followers query methods
The following methods allows to obtain information about the follow chain:

```cs
public static bool HasFollower(AnimIDs followerid)
```
Returns true if any `tempfollowers` has an `originalid` of `followerid` - 1, false otherwise.

```cs
private static EntityControl GetLastFollowing()
```
Returns the last entity in the `player` follow chain or `player`.`entity` if the `player`.`entity` has no `followedby`. This will prioritise the last `playerdata`'s `entity`, but if that one has a `followedby`, the last `map`.`tempfollowers` is returned instead.

```cs
public static EntityControl GetExtraFollower(int id)
```
Returns the first follower entity in `map`.`tempfollowers` whose animid is `id` or null if no such follower exists.
