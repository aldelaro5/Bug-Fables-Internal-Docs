# MusicRange
A spherical radius that when the player enters it, changes the current music playing. The music, amount of frames to wait between distance checks, radius, rate of volume change and max volume are configurable.

## Data Arrays
- `data[0]`: OVERRIDEN (to the value of `data[1]`, serves as an internal cooldown to throttle distance checks with `vectordata[0]`.x)
- `data[1]`: The amount of frames to wait between distance changes of the player and `vectrodata[0].x`
- `data[2]`: The [Music](../../../Enums%20and%20IDs/Musics.md) id this is changing to
- `data[3]`: If it's not -1, the music range Updates are disabled which disables any effects
- `vectordata[0].x`: The max radius of this range relative to its position that will change the music if the player goes within it and change it back it goes outside of it
- `vectordata[0].y`: The rate of the music fading in or out, between 0.0 and 1.0 (it's a volume lerp factor)
- `vectordata[0].z`: The max volume multiplier when fading in the music
- `vectordata[1]`: OVERRIDEN. The x to 0.0 and the y / z to the end and begin loop points respectively of the [music](../../../Enums%20and%20IDs/Musics.md) whose id is `data[2]`

## [CreateEntities](../../EntityControl/CreateEntities.md)
The `musicrangeanim` of the map is set to the map entity id being created here. This means the last music range defined in the map will be allowed control the volume of the main music playing in that music range's Update.

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled
- entity.`activeonpause` is set to true
- `data[0]` is overriden to `data[1]`
- entity.`sound` properties are initialised:
  - The clip is set to one loaded using the [Music](../../../Enums%20and%20IDs/Musics.md)'s name at id `data[2]`
  - The volume to 0.0
  - The loop to true
  - Play is called on it
- `vectordata[1]` is overriden to have its x be 0.0 and its y / z be the end and begin loop points respectively of the corresponding [music data](../../../TextAsset%20Data/Musics%20data.md#musics-data) whose music id is `data[2]`

## Update
Updates are disabled if `data[3]` exists and isn't -1.

The temporary field `data[0]` serves as a frame countdown initialised from `data[1]`. It decrements each update with the `hit` (whether or not the player is in the music range) staying at false until it goes to 0. When that happens, a distance check is peformed with this music range's position and the player's. If the distance is less than `vectordata[0].x`, then `hit` is set to true, false otherwise. 

When `hit` is true, `inmusicrange` is set to the `mapid` and a CheckSamira is perfomed to add the song to her list when applicable. 

When `hit` is false, `inmusicrange` is set to -1 if it was the `mapid`. In either case, the `data[0]` countdown is reset to `data[1]` and this cycle repeats for each update.

No matter what, some more logic follows. If the `inmusicrange` is `mapid` by then (we need to play the music) and the entity's `sound` isn't playing, Play is called on it.

The volume is set to be a lerp from the existing entity.`sound` volume to either 0.0 if we aren't playing or to the appropriate game volume settings * `vectordata[0].z` with a factor of `vectordata[0].y`.

If the current map's `musicrangeinanim` is the `mapid` (meaning this is the music range that is allowed to control the main music volume), the first `music`'s volume is set using a lerp from its exiting volume to either the musicvolume if we aren't playing the song or to 0.0 if we are with a factor of `vectordata[0].y`. This does the opposite of what we would have done to the song volume by fading out the main music if it's playing or to fade it in if the music is to stop playing.

## FixedUpdate
This is the only object type with logic in FixedUpdate.

If the entity.`sound` is playing and the playback time reached `vectordata[1].y` (the music end loop point) or further, the playback time will be set to `vectordata[1].z` (the music begin loop point).

## EntityControl.[LateUpdate](../../EntityControl/Update%20process/Unity%20events/LateUpdate.md)
For any inactive LateUpdates, objects of this type aren't allowed to have their `sound` volume set to 0.0

## EntityControl.[UpdateSound](../../EntityControl/Update%20process/UpdateSound.md)
The entire logic from this update method is excluded for any objects of this type. Combined with the LateUpdate exclusion mentioned above, it means this object cannot perform any entity LateUpdate sounds logic.