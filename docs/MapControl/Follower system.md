# Follower system
Maps can entirely control the follower systems which is the system that dictates which followers is allowed to follow the player. It also manages the presence of `chompy` as a follower. The general function of the system is documented in the game's followers system documentation, but since MapControl plays an integral part in configuring and controlling it, the MapControl specific parts are documented here.

The system can be configured on the map with the following configuration fields:

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|canfollowID|int[]|The [animIDs](../Enums%20and%20IDs/AnimIDs.md) that are allowed to follow the player party on this map when present in instance.`extrafollowers`. This restriction is ignored if the GameObject's name is `0` (which is normally only the case for the `TestRoom` [map](../Enums%20and%20IDs/Maps.md))|Empty array|
|followerylimit|float|The maximum following distance (absolutely value) in y allowed for any entity following another. If the y distance between the entity and its followee gets higher than this value and we aren't in a `minipause`, the [DoFollow](../Entities/EntityControl/Notable%20methods/Follow.md#dofollow) of the entity will teleport them to their followee instantly. This should NEVER be negative because the distance is meant to be expressed as an absolute value|20.0|
|closemove|bool|This field if true allows the map to force the CloseMove entity follow logic of the player party. NOTE: The influence of this fiels is extremely complex and inconsistent, primarily during the `BanditHideout` stealth section, more details can be found in the section below|false|

## MapControl's involvement in the follower system
There's an important distinction between the player requesting an entity to follow the party and the MapControl allowing said followers. They are actually managed at 2 different places:

- instance.`extrafollowers` tells the [animIDs](../Enums%20and%20IDs/AnimIDs.md) that wants to follow the player party. This array is saved on the [save file](../External%20data%20format/Save%20File.md)
- instance.`map`.`canfollowID` tells which [animIDs](../Enums%20and%20IDs/AnimIDs.md) are allowed to follow the party. Since it's a configuration field, it is meant to have its value come from prefab, but [AreaSpecific](Init%20methods/AreaSpecific.md) can do modifications on it if needed

This distinction is important because it means that even if instance.`extrafollowers` contains an animid and even if this information is loaded from the save file, the current map has the final say on allowing or denying the following. If the animid isn't in its `canfollowID`, the entity won't appear and no following will happen. It's essentially an allow list system.

The only exception to this rule is when the GameObject's name of the MapControl is `0`. Since this name always corresponds to the [map](../Enums%20and%20IDs/Maps.md) id, it means it should only apply to the `TestRoom` map. On that map specifically, the `canfollowID` rule doesn't apply and any animid in instance.`extrafollowers` will be added as a follower.

The followers that are actually present are stored in the map's `tempfollowers`. This array excludes the main player party member, but it does include `chompy` or any other followers.

## MapControl's flow for adding followers
While AddFollower can be called manually well after the map is loaded, MapControl can call it in its first LateUpdate (known as `latestart`). This is the part that enforces the `canfollowID` allow list, but also manages the presence or absence of `chompy`

For adding any followers other than `chompy`, here's how this process goes:

- As mentioned earlier, if the [map](../Enums%20and%20IDs/Maps.md) is `TestRoom`, AddFollower is called for each instance.`extrafollowers` as the id and null as the caller so every follower the player has is allowed
- Otherwise, only the ones present in `canfollowID` will be added as followers in the same way. After AddFollower is called, the following is done:
    - The follower's `tempfollowerid` is set to the matching `canfollowID` index. NOTE: This is incorrect, see the section below for details on the `tempfollowerid` issues
    - If the follower's [animid](../Enums%20and%20IDs/AnimIDs.md) is `Maki`, its `ccol` gets a height of 3.0 and a center of (0.0, 1.5, 0.0)
    - The follower's `onground` is set to false (this is to force an animation update)

### Issues with followers's `tempfollowerid`
An entity's `tempfollowerid` field is intended to express the index of the follower in the map's `tempfollowers` array. It is tracked purely for EntityControl to perform anti z fighting measures during the [Follow logic](../Entities/EntityControl/Notable%20methods/Follow.md) when the `player` is `flying`.

The problem is this isn't always assigned correctly because AddFollower doesn't do this so it relies on the caller to do it and not all callers do it correctly. In the case of the `TestRoom` map, it's not assigned at all so z figthing can happen when the `player` is `flying` still.

More confusingly however is the normal case of adding followers other than `chompy`: in this case, MapControl incorrectly sets the follower's `tempfollowerid` to the index of the matching animid found in `canfollowID`. The problem is `canfollowID`'s order isn't guaranteed by this point to give matching indexes because an unknown amount of followers might not have existed in instance.`extrafollowers`. What this means is that it's possible that 2 followers ends up with the same `tempfollowerid` which would cause them to z fight when the `player` is `flying`. Whether or not this can happen depends very specifically on the order of `canfollowID` as specified in the map's prefab (after [AreaSpecific](Init%20methods/AreaSpecific.md) modified it if needed) and which specific animids are present in instance.`extrafollowers`.

### Issue regarding follower order consistency
There's another different issue the follower system has, but it's more a design issue than a programming mistake: the order followers are in can change between the moment the follower is added and the next map load.

This is because AddFollower ALWAYS append the follower to the end of `tempfollower` and it always becomes the last entity in the follow chain. This isn't consistent with how MapControl builds this chain on `latestart`: it adds the requested ones in the order they appear in `canfollowID` and then adds `chompy` if needed.

It means for example that a follower could be following `chompy` when added after the map's `latestart` (which is frequently the case in events), but when loading a map right next to the current one, the follower won't be following `chompy` and rather, `chompy` will be the last follower. This inconsistency can be unexpected and since it's not possible to tell where the follower should be inserted in the chain, it can always happen when calling AddFollower after the map's `latestart`.

### `chompy` following
`chompy` is a special kind of follower because it's not part of the main player party, but a regular `tempfollowers` that simply keeps reappearing on each map load. This is acomplished by logic in the first LateUpdate (known as `latestart`) that happens right after the main followers logic described above happens.

It only happens if [flag](../Flags%20arrays/flags.md) 402 is true (Chompy is with Team Snakemouth) and the `player` isn't in a `submarine`. If these conditions are met, the following happens:

- AddFollower is called without a caller and with 169 (`ChompyChan`) as id
- The map's `chompy` is set to the follower entity just added
- `chompy`'s `tempfollowerid` is set to the matching index of `tempfollowers`
- `chompy`'s `onground` is set to false (this is to force an animation update)

## Configuring the entity's [follow logic](../Entities/EntityControl/Notable%20methods/Follow.md)
There are 2 ways MapControl can configure the actual follow logic entities have: `followerylimit` and `closemove`.

### `followerylimit`
This configuration field controls the maximum y distance followers are allowed to be from their followee before they get teleported to their followee. It's enforced by EntityControl's [DoFollow](../Entities/EntityControl/Notable%20methods/Follow.md#dofollow).

It should be noted that there's another entity field involved in this teleport logic: the entity's `followlimit` field. This field is almost always 20.0 with the only exception being the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) entity where it is 6.0. That one enforces the maximum square distance allowed in x/z before the teleportation happen.

They both enforce the same thing, but only one limit being violated is enough for the teleport to occur.

### `closemove`
The entity following logic has a different mode of operation called CloseMove. More details can be learned in the [follow move logic](../Entities/EntityControl/Notable%20methods/Follow.md#decide-whether-to-move) documentation, but it's essentially a method on EntityControl that if it returns true, it will cause entities to follow their followee twice as close than normal.

The map's `closemove` configuration field is involved in the CloseMove check, but in a rather complex fashion. Having this field set to true may implicate the CloseMove logic, but it's not enough on its own to enforce it.

First off, no matter what the value of this field is, CloseMove requires that that the `player` exists and isn't `dashing` for the logic to be used. If these conditions are met, then the logic still apply if the `player`'s `forceclosemove` is true no matter what the map's `closemove` says. Additionally, if the entity isn't part of the `mainparty` then whether or not the logic apply only depends on the `player`'s `forceclosemove` so the map's `closemove` isn't involved. Also, even if it is part of the `mainparty`, the `player` still needs to be rooted to the scene as otherwise, it entirely depends on the `player`'s `forceclosemove` once again (the `player` is normally rooted, but it can be on certain platform that childs the `player` to them). Finally, even if all of the above don't apply, but the map's `closemove` is true, then [flags](../Flags%20arrays/flags.md) 401 (the "stealth mode" flag) needs to be true as otherwise, it once again entirely depends on the `player`'s `forceclosemove`.

To summarise, the configutation field only has an impact if all of the following are true:

- The `player` exists and isn't `dashing` (otherwise, the logic never applies)
- The entity is part of the `mainparty` (otherwise, the `player`'s `forceclosemove` decides everything which ignores the map's `closemove`)
- The `player` is rooted to the scene (othereise, same outcome: the `player`'s `closemove` decides everything)
- The [flag](../Flags%20arrays/flags.md) 401 is true (otherwise, same outcome: the `player`'s `closemove` decides everything)
- The `player`'s `forceclosemove` is false (otherwise, the configuation field wouldn't change that the logic would apply since `forceclosemove` takes precedence over it)

What's effectively happening is that in order to force the CloseMove logic to happen, there's 2 ways:

- The `player`'s `forceclosemove` is true: this takes precedence over everything such that CloseMove always applies unless either the `player` doesn't exist or it is `dashing`. In this case, the map's `closemove` is effectively ignored. However, it should be noted that under normal gameplay, this only happens when the `player` is on top of a [TempPlatform](../Entities/NPCControl/ObjectTypes/TempPlatform.md)
- If the above doesn't apply, BOTH the map's `closemove` AND [flag](../Flags%20arrays/flags.md) 401 needs to be true to guarantee that the CloseMove logic will happen alongside the other conditions mentioned further above

The flag 401 check is what makes the influence of the map's `closemove` more complicated because they are both managed very inconsistently with each other.

#### Exploring further the issue with `closemove`
While `closemove` is a configuration field, it can be overriden by [AreaSpecific](Init%20methods/AreaSpecific.md):

- It is always overriden to true in the `WaspKingdom` [area](../Enums%20and%20IDs/librarystuff/Areas.md) except for a list of specific maps. In this case, [flag](../Flags%20arrays/flags.md) 401 (and also `cantcompass`) will be set to true if `closemove` was also overriden to true. Therefore, the CloseMove logic always consistently apply in the `WaspKingdom` except for the following maps (it covers all maps used in the stealth section during chapter 5):
    - `WaspKingdomOutside`
    - `WaspKingdomJayde`
    - `WaspKingdomMainHall`
    - `WaspKingdomPrison`
    - `WaspKingdom5`
    - `WaspKingdomQueen`
    - `WaspKingdomThrone`
- In the `GiantLair` [area](../Enums%20and%20IDs/librarystuff/Areas.md), `closemove` isn't overriden, but [flag](../Flags%20arrays/flags.md) 401 has its value set to it. The maps in this area correctly have `closemove` defined to true when needed so it means this stealth section is consistently forcing the CloseMove logic to apply
- [flag](../Flags%20arrays/flags.md) 401 is always overriden to false in any areas other than `WaspKingdom` or `GiantLair`. This is consistent with the above, but it's not with what is explained below

There is one stealth section in the game where the CloseMove logic isn't applied consistently: the `BanditHideout` [area](../Enums%20and%20IDs/librarystuff/Areas.md). This one has the following management:

- [Event](../Enums%20and%20IDs/Events.md) 109 will set [flag](../Flags%20arrays/flags.md) 401 to true at the `HideoutCentralRoom` [map](../Enums%20and%20IDs/Maps.md) towards the end of the event and set it back to false at the same event in `HideoutWestStorage`
- `closemove` is set to true on all the `BanditHideout` [maps](../Enums%20and%20IDs/Maps.md) contained in the stealth section

This should mean that the CloseMove logic should be consistent in this section. The problem is [AreaSpecific](Init%20methods/AreaSpecific.md) overrides [flag](../Flags%20arrays/flags.md) 401 to false at `BanditHideout` because it's not correctly included in that logic where its value shouldn't change there. This leads to the unexpected behavior that while the CloseMove logic applies right when event 109 ends, it will immediately stop to apply the moment another map is loaded from `HideoutCentralRoom`. This can be seen by observing the party's follow distance incorrectly changing.
