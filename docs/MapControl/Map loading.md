# Map loading
There's only at most 1 MapControl that can be loaded at once and it will be the current map assigned to instance.`map`.

It means that in order to go to another map, the current one needs to be destroyed before instantiating the new one. Since this game doesn't involve traditional Unity scene loading, all of this is done through code. This implies the game is entirely managing the map loading system by itself.

There's only one way to safely change the current map: via the LoadMap method. This method however only does the map unloading/loading part. To have the complete transition done when using the [transfer](../SetText/Individual%20commands/Transfer.md) SetText command or when going through a [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md), the TransferMap coroutine is used which ends up calling LoadMap, but with scripted party movement and fading transitions.

Basically, to have the complete map loading transition, the TransferMap coroutine is used and for any kinds of scripted map loading, LoadMap is called directly. It is very common for [events](../Enums%20and%20IDs/Events.md) to prefer the latter option because they can control the game so they can do the loading transition by themselves and simply call LoadMap directly. Both LoadMap and TransferMap are in MainManager.

## LoadMap
There are 3 overloads, but all of them ends up at the first one:

(1) The main method that all overloads and TransferMap ends up using. The `id` parameter is the int representation of the new [map](../Enums%20and%20IDs/Maps.md) value to change the current map to. If this value is the current `map`'s id, it will be reloaded.

```cs
public static void LoadMap(int id)
```

(2) which calls (1) where `id` is `map`.`name` which should always be the matching [maps](../Enums%20and%20IDs/Maps.md) of the current map. Effectively, it reloads the current map to the same map.

```cs
public static void LoadMap()
```

(3) This calls (1), but before doing so, if `recreateplayers` is true, all `playerdata` entity gets destroyed and the `player` is set to null. This will have the (1) overload recreate the `player` and the `playerdata` entities from scratch.

```cs
public static void LoadMap(int id, bool recreateplayers)
```

Here's what the method does:

### Setup
- The camera angling logic is reset by setting `changecamspeed` and `camanglechange` to false and setting `camanglespeed` to 0.1
- [flag](../Flags%20arrays/flags.md) 400 (temporary global flag) is reset to false

### Current map destruction
This section only happens if instance.`map` isn't null and it involves its destruction:

- All `playerdata` (if they exists) gets rooted to the scene and their `followedby` reset to null. Effectively, it breaks the follow chain and makes sure they not childed to anyone in the map
- All of the instance.`map`.`tempfollowers` gets destroyed
- instance.`map` gets destroyed
- If not `inevent`, the existing ressources are cleaned up by doing the following:
    - GC.Collect is called. NOTE: This doesn't actually do anything useful because of the method call right after this
    - Ressources.UnloadUnusedAssets is called. Since this ends up calling GC.Collect, the above call didn't do anything useful

### New map load
This section always happen:

- The new map gets instantiated using the prefab located at `Ressources/Prefabs/Maps/X` where `X` is the string representation of the [map](../Enums%20and%20IDs/Maps.md) value whose int representation is the id parameter of the method
- `map` gets set to the MapControl of the newly instantiated GameObject and its name is set to the string representation of the id parameter (it means the GameObject's name is an integer string representing the [map](../Enums%20and%20IDs/Maps.md) value)

### `playerdata` recreation
If `player` is null which can only happen if overload (3) was used with true as recreateplayer, SetPlayers will be called since the `playerdata` needs to be created as they just got destroyed by this point. This is what the method does on each `playerdata`:

- `entity` is set to a new entity created vis [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with name being `Player X` where `X` is the `playerdata` index
- `entity`'s [animid](../Enums%20and%20IDs/AnimIDs.md) is set to the `playerdata`'s `animid` which is actually the fixed party order id, but they are aligned here to match the animids
- `entity`'s `alwaysactive` is set to true since it's a player entity that shouldn't be inactive
- `entity`'s [conditions](../Battle%20system/Actors%20states/Conditions.md) is reset to a new empty list
- If it's the first `playerdata`, it's the main player:
    - A PlayerControl is added to the `entity`'s GameObject
    - instance.`camtarget` is set to the `entity`'s transform so the camera follows it
    - `entity`'s `tag` is set to `Player`
- Otherwise, it's a player follower:
    - `entity`'s `following` is set to `playerdata[0]`.`entity` if it's the second or to `playerdata[1]`.`entity` if it's the third so it setups the follow chain where `playerdata[2]` follows `playerdata[1]` which follows `playerdata[0]` (the main player)
    - `entity`.`followoffset` is set to the index * 0.2 to prevent z fighting
    - `entity`'s GameObject layer is set to 9 (`Follower`)
    - The z position of the `entity` is incremented y its `followoffset` which further prevents z fighting
    - `entity` tag is set to `PFollower`
    - `entity` becomes part of the `mainparty`
    - The previous `playerdata`'s `entity`.`followedby` is set to this `entity` which resolves the follow chain in reverse order

### ForceLoadSprites
This section always happen:

- ForceLoadSprites is called which starts a coroutine called FLS which will doe the following in paralel(assuming the `map`.`entities` isn't null):
    - All the `map`.`entities`.`sprite`.sprite gets assigned to new dummy GameObjects's SpriteRenderer created just for this. They are childed to the `map` and positioned at the `player`
    - A fame gets yielded
    - All the dummy GameObjects created earlier gets destroyed

### `player` cleanup
This section only happens if `player` isn't null meaning that the overload (3) wasn't used (while the `playerdata` are recreated by this point, the PlayerControl hasn't started yet so `player` is still null here):

- `player`'s `entity`.`sound` is set to stop looping and stops playing
- `player`.`pausecooldown` is set to 120.0 so no pausing is allowed for the next 120.0 frames

## TransferMap
There are 3 overloads, but all of them ends up at the first one:

(1) The main method that all overloads ends up using. Here are the parameters:

- `targetmap`: The int representation of the [maps](../Enums%20and%20IDs/Maps.md) value corresponding to the new map to change to
- `moveto`: The position the party will go towards before laoding the map
- `tppos`: The position the party will be placed after loading the map
- `othermovepos`: The position the party will go towards after loading the map and after they were placed at `tpppos`
- `caller`: The [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) that is configuring this map transfer. This is optional, if it's null, no configuration from one will happen

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos, NPCControl caller)
```

(2) Starts a coroutine with (1) where `caller` is null then yield null. This is only used by the [transfer](../SetText/Individual%20commands/Transfer.md) SetText command which simulates a transfer, but there's no [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) involved.

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos)
```

(3) Starts a coroutine with (1) where `moveto` is the `player` position, both `tppos` and `othermovepos` are the sent `targetpos` and `caller` is null, then yield null. This overload is UNUSED.

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 targetpos)
```

What follows is what the coroutine does.

### Setup

- `roomtransition` is set to true which prevents DoClock from cleaning up unused ressources
- If there is a caller [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md), its `data`, `emoticonoffset.x` and `vectordata[3]` through `vectordata[6]` are saved locally since they are the data fields that configures this map transfer
- All of the current `map`'s `entities` has [BreakIce](../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on them and their `npcdata`'s `boxcol` disabled if it exists
- We enter a `minipause`
- CancelAction is called on the `player`
- Yield all frames until `pausemenu` gets null (meaning the game is unpaused)

### Move out and fade in

- PlayTransition is called with id and data set to 0 (fade out) at a speed of 0.1 and a color of pure black
- If there is a caller [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) and its `data[4]` isn't 1, it means the movement during the fade in will happen:
    - `player` [MoveTowards](../Entities/EntityControl/EntityControl%20Methods.md#movetowards) moveto at 1.0 speed using 1 (`Walk`) as state and 0 (`Idle`) as stopstate with ignore_y
    - instance.`camtargetpos` is set to the `player` and instance.`camtarget` is set to null so it fixes the camera in place to where the `player` is
    - Yield all frames until the `player`'s `forcemove` is done
- Otherwise (the caller's `data[4]` is 1), no movment will happen and instead, all `playerdata` has their `rigid` gravity disabled and put in isKinematic followed by a frame yield
- Yield all frames until the color of the `transitionobj[0]`'s SpriteRender gets to 0.9 or above (it basically yields until the fade out is 90% done)
- `transitionobj[0]`'s SpriteRender's color alpha is set to 1.0 (fully opaque). This forces the end of the PlayTransition call from earlier because it's been waiting for that to happen
- Yield for a frame

### Map load and move in preparation

- LoadMap is called with the targetmap as new map id
- If we aren't instance.`inevent`, instance.`insideid` is set to -1 which marks the player not being in any [inside](Insides.md)
- instance.`globalcooldown` set to 30.0 which prevents most inputs for 30.0 frames
- instance.`inmusicrange` is set to -1 which clears any [MusicRange](../Entities/NPCControl/ObjectTypes/MusicRange.md) from the previous map if it existed
- Yield for a frame
- `MainCamera` position is manually set to `playerdata[0]` position + instance.`camoffset`
- instance.`camspeed` is set to 1.0, but its existing value is saved for restoring it later
- Yield for a frame
- All `playerdata`'s `entity` has the following happen on them:
    - position set to tppos + `MainCamera`'s forward vector * the `playerdata` index / 10.0 (this essentially prevents z fightings)
    - `rigid` velocity zeroed out with gravity and without isKinematic
    - `onground` set to false (forces an animation update)
- Yield for a frame
- All `map`'s `entities` (the new map as it's loaded by this point) has [CheckNear](../Entities/EntityControl/EntityControl%20Methods.md#check-near) called on them
- Yield for a frame
- ForceLoadSprite is called which does the same than what was already done in LoadMap so this doesn't do anything useful
- Yield for 0.1 seconds
- All `playerdata` [FaceTowards](../Entities/EntityControl/EntityControl%20Methods.md#facetowards) othermovepos
- instance.`camtarget` is set to the `player` so the camera is back to following the `player`
- If there was a caller [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md), this is where its configuration fields are used to configure the camera which will override the ones used for the new [map camera system](Camera%20system.md):
    - If `data[1]` is 1, instance.`camoffset` is set to `vectordata[3]`
    - If `data[2]` is 1, instance.`camangleoffset` is set to `vectordata[4]`
    - If `data[3]` is 1, `map`.`camlimitpos` is set to `vectordata[5]` and `map`.`camlimitneg` is set to `vectordata[6]`
- Yield for 0.3 seconds
- instance.`camspeed` is reset to its value saved earlier
- Yield for a frame

### Fade in and move in

- PlayTransition is called with id and data set to 1 (fade in) at a speed of 0.1 and a color of pure black
- Yield for a frame
- If there was a caller [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) and its `emoticonoffset.x` was above 0.1, the player party needs to be moved via a jump:
    - `player`'s `jumproutine` is set to a new JumpTo call towards othermovepos with a height of the caller's `emoticonoffset.x`
    - Yield until the `player`'s `jumproutine` is done. Before each frame yield, all `playerdata`'s `entity`'s [animstate](../Entities/EntityControl/Animations/animstate.md) is set to 3 (`Jump`) and their `onground` to false (force an animation update)
- Otherwise (normal movement without jump):
    - `player`'s `entity` [MoveTowards](../Entities/EntityControl/EntityControl%20Methods.md#movetowards) othermovepos at a speed of 1.0 using 1 (`Walk`) as state, 0 (`Idle`) as stopstate and with ignore_y
    - Yield all frames until the `player`'s `forcemove` is done

### Cleanup

- `player`'s `entity` has DetectIgnoreSphere called with ignore meaning the player entity's `detect` is set to ignore all collision with any NPCControl's `scol`
- We exit the `minipause`
- `player`'s `lastpos` and `lastloadzone` are set to their current position
- Yield for a frame
- `player`'s `entity`.`hitwall` is set to false forcing its [wall detector](../Entities/EntityControl/EntityControl%20Methods.md#detector) off
- `player`'s `npc` is reset to a new empty list
- `player`'s `pausecolldown` is set to 7.0 so no pausing can happen for the next 7.0 frames
- `roomtransition` is reset to false which allows DoClock to cleanup unused ressources again
