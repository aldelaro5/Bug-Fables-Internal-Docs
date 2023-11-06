# MusicRange
A spherical radius in which, when entered, changes the current music playing. The music, amount of frames to wait between distance checks, radius, rate of volume change and max volume are configurable.

## Data Arrays
- `data[0]`: Overriden to `data[1]`
- `data[1]`: The amount of frames to wait between `hit` checks (Moved to `data[0]`)
- `data[2]`: The [Music](../../../Enums%20and%20IDs/Musics.md) id this is changing to
- `data[3]`: If it's not -1, disables any Update ???
- `vectordata[0].x`: The max distance the player can be away from this for `hit` to remain true, `hit` is false otherwise
- `vectordata[0].y`: The rate of the music fading in or out, between 0.0 and 1.0 (it's a lerp factor)
- `vectordata[0].z`: The max volume multiplier when fading in the music
- `vectordata[1]`: ??? (overriden to (0.0, `musicloop[data[2]][0]`, `musicloop[data[2]][1]`))

## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

The entity's `activeonpause` is set to true.

Then, `data[0]` is overriden as mentioned above, same for `vectordata[1]`.

Finally, the entity's `sound` properties are initialised:
- The clip is set to one loaded using the [Music](../../../Enums%20and%20IDs/Musics.md)'s name at id `data[2]`
- The volume to 0.0
- The loop to true
- Play is called on it

## Update
Updates are disabled if `data[3]` exists and isn't - TODO: why ???

First, The temporary field `data[0]` serves as a frame countdown initialised from `data[1]`. It decrements each update with the `hit` staying at false until it goes to 0. When that happens, a distance check is peformed with this object's position and the player's. If the distance is less than `vectordata[0].x`, then `hit` is set to true, false otherwise. When `hit` is true, `inmusicrange` is set to the `mapid` and a CheckSamira is perfomed to add the song to her list when applicable. When `hit` is false, `inmusicrange` is set to -1 if it was the `mapid`. In either case, the `data[0]` countdown is reset to `data[1]` and this cycle repeats for each update.

No matter what, some more logic follows. If the `inmusicrange` is `mapid` by then (we need to play the music) and the entity's `sound` isn't playing, Play is called on it.

The volume is set to be a lerp from the existing entity's `sound` volume to either 0.0 if we aren't playing or to the appropriate game volume settings * `vectordata[0].z` with a factor of `vectordata[0].y`.

Lastly, if the current map's `musicrangeinanim` is the `mapid`, the first `music`'s volume is set using a lerp from its exiting volume to either the musicvolume if we aren't playing the song or to 0.0 if we are with a factor of `vectordata[0].y`. This does the opposite of what we would have done to the song volume by fading out the main music if it's playing or to fade it in if the music is to stop playing. TODO: what is the difference between `MainManager.instance.inmusicrange` and `MainManager.map.musicrangemain` ???
