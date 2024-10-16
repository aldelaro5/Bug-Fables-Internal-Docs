# Map music system
MapControl extends the management of the [Music playback](../General%20systems/Music%20playback.md) in such a way that maps can predeclare a set of AudioClip and have a way to conditionally select the initial one to play when loaded.

It involves the following configuration fields:

|Name|Type|Description|Default|
|---:|----|----------|-------|
|music|AudioClip[]|The list of AudioClip this map has access to play as background music. If the array is empty, the current music will fade int silence and stop playing|Empty array|
|musicid|int|The default `music` index to use as the background music, but it can be overriden if `musicflags` isn't empty. If `musicflags` isn't empty, it defaults to -1 before having the value be determined by `musicflags` (or stays at -1 if no element is selected). Regardless of `musicflags`, if the value ends up at -1, the current music will fade into silence before it stops playing|0 (-1 if `musicflags` isn't empty)|
|keepmusic|bool|When this is true and instance.`inevent` is false, [Music](Init%20methods/Music.md) won't be called by Start which means the current music playing won't change and `musicid`'s value won't be changed. Essentially, it prevents the music that was playing before loading this map from changing when loading this map|false|
|musicflags|Vector2Int[]|A list of directives to follow regardinig the assignement of `musicid` in the [Music](Init%20methods/Music.md) method. When not empty, `musicid` defaults to -1 followed by each element being processed in reverse order. The first one whose x component is either -1 or a [flags](../Flags%20arrays/flags.md) slot that is true will be selected. When selected, `musicid` gets set to the y component's value and it will be used as the current music. If no suitable element is selected, `musicid` stays at -1 and the current music will fade into silence before it stops playing|Empty array|

Essentially, `music` contains the AudioClip to use as the map's musics and they must be assigned from prefab (they also must reside in the `Audio/Music/` ressources directory so the game can reconcile its [Musics](../Enums%20and%20IDs/Musics.md) enum value).

`musicid` dictates the `music` index to play from the [Music](Init%20methods/Music.md) method (which is called by Start if `keepmusic` is false). It can either have its value directly come from prefab or it can be determined by a list of directives contained in `musicflags` which can conditionally select the `musicid` depending on [flags](../Flags%20arrays/flags.md). It's possible both are left as default in which case, it defaults to `music[0]`.

## Changing `musicid`
This system however doesn't force the music to always be `music[musicid]`, it's possible via 2 of the [ChangeMusic](../General%20systems/Music%20playback.md#changemusic) overloads to change the music after the fact, but based on the map's `music` array.

The parameterless overloads will change to `music[musicid]` if `musicid` isn't negative or to null (silence, stops playing the current music) if either the map is currently loading or `musicid` is negative.

There is also the overload that takes a mapid integer parameter. This one allows to change the music to another `music` element by specifying its index, but without changing `musicid`. It essentially allows to play an alternative music while still leaving the default one as is.

Additionally, it's possible to call the [Music method](Init%20methods/Music.md) directly since it is public after setting `musicid` which would change the default and change the music accordingly.

## About [insides](./Insides.md) transitions
A [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) can specify a [Musics](../Enums%20and%20IDs/Musics.md) id to change to when entering the inside. In that case, [MoveInside](Insides.md#moveinside) will call [ChangeMusic](../General%20systems/Music%20playback.md#changemusic) appropriately.

However, there is an issue when it comes to exiting the inside. When exiting, as long as `music` isn't empty (it normally isn't), ChangeMusic is called with `music[0]`. The problem is this isn't correct: it's possible `musicid` wasn't 0 when the inside was entered. If this happens, the game will STILL play `music[0]` when exiting any inside. This happens regardless if the above music transition feature is used or not.

In practice, this issue doesn't do anything because there are no maps with multiple `music` elements where `music[0]` wouldn't play when going through a DoorSameMap that have a `data[1]` defined.
