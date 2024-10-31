# Music playback
This game offers many ways to configure how and what background music to play. A background music is meant to be an AudioClip with loop points defined.

All music playback goes through a single audio source which is stored in MainManager.`music`. This means only one music can play at a given time and it has to go through this AudioSource because the game manages the music system using it. It differ from a sound which is meant to be played once and there might be multiple of them.

This page talks about the music system in general, but a part of this system is managed by [MapControl's map music system](../MapControl/Map%20music.md). Check the documentation there to learn more about MapControl's influence on the music system.

## ChangeMusic
The only way to play a music or to stop playing the one currently playing is via the MainManager.ChangeMusic method which there are several overloads to choose from.

This overload is where all other overloads ends up using:

(main)
```cs
public static void ChangeMusic(AudioClip musicclip, float fadespeed, int id, bool seamless)
```

### Parameters

- `musicclip`: The AudioClip corresponding to the music to play. If this is null, the method will instead stop the current music and leave it stopped
- `fadespeed`: A multiplier that controls the speed at which the existing music will fade to silence if a music was playing. It corresponds to a multiplier applied to `framestep` and the result is used as a lerp factor of the volume into silence. However, this resulting volume is still scaled accordingly with MainManager.`musicvolume` which is a game setting so both values stacks multiplacitively with each other
- `id`: This is supposed to represent the index of `music` used to change the music playback, but in practice, only the value 0 is valid. This is because `music` always has a single element in its array. Any value other than 0 will cause an exception to be thrown
- `seamless`: If this is true, the fading will be done by doing a crossfade with the existing music instead of a fade out / fade in. This is achieved by temporarily using the first free `sounds` AudioSource where the music will initially play muted. During the fade, as the `music` gets faded out, the `sounds` will fade in by the same inverse proportion. When the cross fade is complete, the playback on `sounds` is stopped which frees it while playback continues at the same time on the `music`. NOTE: This feature assumes that both AudioClip have matching times and are relatively similar with each other

### Overloads

```cs
(1) public static void ChangeMusic();
(2) public static void ChangeMusic(AudioClip musicclip);
(3) public static void ChangeMusic(AudioClip musicclip, float fadespeed);
(4) public static void ChangeMusic(AudioClip musicclip, float fadespeed, int id);
(5) public static void ChangeMusic(int mapid);
(6) public static void ChangeMusic(string musicclip);
(7) public static void ChangeMusic(string musicclip, float fadespeed);
(8) public static void ChangeMusic(string musicclip, float fadespeed, int id);
(10) public static void ChangeMusic(string musicclip, float fadespeed, int id, bool seamless);
```

- (1): Calls (2) where the musicclip is `map.music[map.musicic]` if `map` isn't null (it's not being loaded), `map`.`music` isn't empty and `map`.`musicid` isn't negative. Otherwise, (2) is called with null as musicclip (meaning to stop playing the current music)
- (2): Calls (3) with 0.075 fadespeed
- (3): Calls (4) with 0 as the id (this is essentially the same then calling (4) directly since the id should always be 0)
- (4): Calls (main) with false as seamless
- (5): Calls (2) with `map.music[mapid]`
- (6): If musicclip is null, (3) is called with null as the musicclip and 0.1 as the fadespeed. Otherwise, (4) is called with musicclip being the result of loading the ressource asset at `Audio/Music/X` where `X` is musicclip, 0.1 fadespeed and id of 0
- (7): Calls (4) with musicclip being the result of loading the ressource asset at `Audio/Music/X` where `X` is musicclip and id of 0 (the fadespeed is passed as is)
- (8): Calls (4) with musicclip being the result of loading the ressource asset at `Audio/Music/X` where `X` is musicclip (the fadespeed and id are passed as is). NOTE: This overload is functional, but is UNUSED and calling (7) directly has the same effect because the id should always be 0
- (9) Calls (main) with musicclip being the result of loading the ressource asset at `Audio/Music/X` where `X` is musicclip (every other parameters are passed as is)

(1) and (5) involve the configuration specific to [MapControl map music](../MapControl/Map%20music.md).

Since all of them ends up at (main), this will be what is documented.

### Procedure (main overload)

- If `musiccoroutine` is in progress (a music was playing), the coroutine is stopped as it is about to be set to a new call
- `musiccoroutine` is set to a new SwitchMusic call with the same parameters sent (See the section below for details)
- The matching `musicsids` element (normally always the first one) gets set to a value depending on musicclip:
    - If it's not null, it is set to the integer value of the matching Musics enum value obtained by musicclip.name. It also sets `lastmusic` to the same value
    - Otherwise, it's set to -1

#### SwitchMusic
This is a coroutine with the same signature than the main overload of ChangeMusic and it is what ends up doing the logic of changing the music. It's private in MainManager meaning only ChangeMusic can call it. The coroutine is meant to be stored in `musiccoroutine` if one is in progress (otherwise, the field remains null).

Most of logic happens only if the musicclip isn't the same than the one in `music[id]`'s clip (if it was the same, it means the music is already playing):

- If in seamless mode, the first free `sounds` is picked for the crossfade with the following being setup on it:
    - clip: musicclip
    - time: `music[id]`'s time (this makes the assumption that both AudioClip are similar and have matching times)
    - volume: 0.0 (starts muted)
    - loop: true
    - Play called
- If the existing `music[id]`'s clip isn't null (a music is currently playing), it is faded out. This fading is done by tracking a value from 1.0 towards 0.0 and is considered done when it reaches below 0.045. This is forced stopped if this lasts for more than 500.0 frames. On each frame of the fading, the following happens (once the fading is done, `music[id]` volume gets set to 0.0):
    - `music[id]` volume is set to `musicvolume` * a lerp from the current fading value to 0.0 with a factor of fadespeed (essentially, it progressively gets muted)
    - `music[id]` volume gets decreased by 10% for the following [Musics](../Enums%20and%20IDs/Musics.md) values (checked by parsing the `music[id]` clip name as a `Musics` value):
        - `Theater`
        - `Tension`
        - `Venus`
        - `Battle3`
        - `Bee`
        - `TermiteLoop`
        - `Pier`
    - If in seamless mode, the fade in part of the crossfade is done:
        - The `sounds` volume gets set to `musicvolume` - `music[id]` volume (essentially, as `music[id]` gets faded out, `sounds` volume increases until it reaches `musicvolume`)
        - If the time of the `sounds` and `music[id]` differs by more than 0.15 seconds, the `sounds` time is synced to the `music[id]` one
- Yield for 0.1 seconds
- If musicclip is null, `music[id]` is stopped and its clip set to null
- Otherwise:
    - `music[id]` clip is set to musicclip
    - If in seamless mode, `sounds` gets fully stopped and `music[id]` takes over. `music[id]` time is set to the `sounds` one and `sounds` gets stopped with time set to 0.0 and loop to false
    - Otherwise, (not in seamless), `music[id]` time is set to 0.0 so it resets the playback from the begining followed by a frame yield
    - If `musicresume` is 0 or above (meaning that [StartBattle](../Battle%20system/StartBattle.md) recorded the time to resume the overworld music):
        - `music[id]` time is set to `musicresume`. NOTE: This may be incorrect, see the section below for details on the music playback issue
        - `musicresume` is reset to -1.0
    - `music[id]` volume is set to `musicvolume`
    - `music[id]` plays

Regardless if a new music was played or not, the following is done after:

- If any of the following conditions are true, CheckSamira is called with the `music[id].clip` that's being played:
    - We are in a `battle`
    - We are in instance.`inevent`
    - `map` isn't null (the map isn't being loaded) while its `musicid` is in the bounds of `musicids`
- Yield for a frame
- `musiccoroutine` is set to null which signals the rest of the game that the music switch is complete

### Issue with `musicresume`
This static value is heavily involved in an issue where it's possible the music where this resume feature is used isn't expected. This feature involves the `keepmusicafterbattle` setting where if enabled, this is how it's supposed to work normally:

- On [StartBattle](../Battle%20system/StartBattle.md), as long as the current map contains `music` and its `musicid` isn't negative, battle.`overmusic` will be set to `music[0]`'s time. This records the timestamp to resume the overworld music that's to be played later
- On [ReturnToOverworld](../Battle%20system/Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md), if we aren't instance.`inevent`, it will set `musicresume` to battle.`overmusic`. This essentially slates the next ChangeMusic call to restore the music timestamp, but only on the next call where musicclip isn't null and isn't the same music than the one playing on `music[0]` at the moment. This is meant to be consumed right away because ReturnToOverworld also does a ChangeMusic to the overworld music
- On the next SwitchMusic that applies, `music[0]` time is set to `musicresume` and then `musicresume` gets set back to -1. This not only restores the timestamp that was saved earlier, but it also resets the value so it's not used again until needed

However, there is an edgecase that isn't handled correctly: if map.`nobattlemusic` is enabled on the map, the ChangeMusic call that normally happens on ReturnToOverworld won't consume the `musicresume` value. This is because since the music hasn't changed between the moment StartBattle was called and the moment ReturnToOverworld was called, the ChangeMusic call won't switch anything and the `musicresume` value is left dangling.

The result is the next ChangeMusic call that happens on any musicclip that isn't null and isn't the current `music[0]` clip will have the timestamp restored. 

However, this is unexpected: it's possible musicclip is a different AudioClip. It would mean the game can attempt to resume the playback, but at an incorrect or invalid timestamp. In the case of the latter, a Unity assertion will be logged as error, but it will be gracefully handled by not changing the time property of the AudioClip so the music starts playing from the start. In the case of the former however, it will result in the music playback starting from the middle instead of the start.

## Music looping
The music are designed to be played in loop. The loop points are defined in the [LoopPoints TextAsset](../TextAsset%20Data/Musics%20data.md#looppoints-data). As for how it is done, it involves a private method on MainManager called LoopMusic which is only called by FixedUpdate.

On the first call, `musicloop` gets initialised with the TextAsset data. From there, this method only changes all `music` time with the starting timestamp if the following conditions are fufilled:

- The ending timestamp isn't 0.0
- The `music` time is at or past the ending timestamp

As for how it accesses the `musicloop` element, it's done via `lastmusic` which is only set by ChangeMusic. This allows to address the data by the [Musics](../Enums%20and%20IDs/Musics.md) id that matches the one currently playing.

## CheckSamira
SwitchMusic involves an important method that interacts with music playback and it involves instance.`samiramusics`. This array is saved on the [Save file](../External%20data%20format/Save%20File.md) and it contains all the available songs to purchase from Samira. CheckSamira allows to unlock one when it's being played for the first time by adding it to instance.`samiramusics` if it wasn't present.

However, in practice, 5 [Musics](../Enums%20and%20IDs/Musics.md) are excluded. This is because on [MapControl](../MapControl/MapControl.md)'s Start, another method called FixSamira is called which will always remove these `Musics` from the array if they were present. Here are the `Musics` excluded:

- `Title`
- `Wind`
- `Water`
- `MachineHum`
- `Breathing`
