# Sounds playback
The game has a system made to play short SFX using a pooled set of AudioSources with configurable pich and volume with the optional ability to loop.

These are internally refered to as sounds. A sound unlike a [music](Music%20playback.md) is made to be played using an AudioSource from the `sounds` array. There are 15 of them available on boot and any playback will by default pick the first one that is free. These AudioClip are meant to be played in short burst as needed while a music is meant to play on `music[0]` only and for a longer period. Check the music playback documentation for how music works.

Sounds shouldn't be confused with [bleeps](../SetText/bleep.md) which have their own dedicated AudioSource called `bleep`. Check the bleep documentation to learn more.

All sounds AudioClip are located in the `Resources/audio/sounds` asset directory and they can be identified by their filename.

As for what consititute a free and a reserved `sounds` AudioSource, the method GetFreeSound is what determines this:

```cs
private static int GetFreeSound()
```
It returns the first `sounds` index (between 0 and 14) whose AudioSource's `isPlaying` is false. If every `sounds` elements are playing something, the last `sounds` element is returned even if it is playing an AudioClip.

While it is possible to specify a particular `sounds` index for the playback, specifying one is optional. By default, one will be provided using the return value of GetFreeSound.

## Playing sounds
Every single sound playback is played through a method with a wide variety of overloads available: PlaySound.

```cs
public static AudioSource PlaySound(string soundclip)
public static AudioSource PlaySound(string soundclip, int id)
public static AudioSource PlaySound(string soundclip, int id, float pitch, float volume)
public static AudioSource PlaySound(string soundclip, int id, float pitch, float volume, bool loop)
public static AudioSource PlaySound(string clip, float volume)
public static AudioSource PlaySound(string clip, float pitch, float volume)
public static AudioSource PlaySound(AudioClip soundclip)
public static AudioSource PlaySound(AudioClip soundclip, int id)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch, float volume)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch, float volume, bool loop)
```
It will plays the `soundclip` AudioClip using the `sounds[id]` AudioSource with a pitch of `pitch`, volume of `soundvolume` * `volume` and a loop property of `loop` then returns the AudioSource where the AudioClip started playing as well as setting `lastsoundid` to `id`. In practice, `id` can only be from 0 to 14 as `sounds` has a fixed size of 15 enforced by LoadEssentials.

If SoundVolume returns false (meaning sounds are muted), the method returns null instead. NOTE: It does NOT return an empty AudioSource, this means the return value needs to be null checked to avoid an exception being thrown.

If `id` is -1, the value to use as the id is determined automatically using the GetFreeSound method.

For the overloads taking a string instead of an AudioClip, `soundclip` becomes the result of loading the AudioClip at `Audio/Sounds/X` where `X` is the string provided.

For overloads not taking every parameters, here are their default values:

- `id`: -1
- `pitch`: 1.0 (normal)
- `volume`: 1.0 (normal)
- `loop`: false

There is also a method that allows to call PlaySound after a delay: LateSound.

```cs
public static IEnumerator LateSound(string sound, float delay)
```
It will yields for `delay` amount of seconds before calling PlaySound(`sound`) which plays the sound whose AudioClip is named `sound`.

Additionally, there's also a managed way to call PlaySound using a sound that is bound to a specific animid called their move sound:

```cs
public static void PlayMoveSound(EntityControl entity)
public static void PlayMoveSound(int animid, int soundid, bool fly)
```
It either calls PlaySound using `id` as the id with loop or does nothing depending on the `animid` / `entity.originalid` and the value of `fly`. If PlaySound is called, the soundclip, pitch and volume are determined depending on the `animid` / `entity.originalid` where the sound represents the appropriate movement sound to play whenever any entity whose animid is `animid` moves.

The overload with just an `entity` parameter has the `soundid` value default to 9 and `fly` value default to whether or not `entity`.`height` is higher than 0.1 (meaning they have a visual offset in y).

Here are the specific properties for each supported value of `animid` (if the value isn't in this list, PlaySound is not called and the method does nothing):

- `Scorpion`: `Step` with 1.0 pitch and volume
- `DivingSpider`: `WetStep` with 1.0 pitch and volume
- `FlyTrap` and `Krawler`: `Scuttle3` with 1.0 pitch and volume
- `Armorpillar`: `FunnyStep` with 0.9 pitch and 0.3 volume
- `Mushroom`: If `fly` is true, `Fly` with 0.5 pitch and 1.0 volume, no PlaySound call otherwise
- `Seedling` and `Flowering`: If `fly` is true, `Toss2` with 1.1 pitch and 0.5 volume, no PlaySound call otherwise
- `Thief`, `Midge` and `Mantidfly`: If `fly` is true, `BugWing` with 1.0 pitch and volume, no PlaySound call otherwise
- `Spuder`: `Scuttle2` with 0.9 pitch and 0.8 volume
- `MimicSpider`: `Scuttle2` with 1.1 pitch and 0.8 volume
- `Plumpling`: `ThumpSoft` with 0.65 pitch and 0.65 volume
- `CordycepsAnt`: `Scuttle` with 0.5 pitch and 0.8 volume
- `Abomihoney`: `PuddleMove` with 1.15 pitch and 1.0 volume

Finally, here are other helper methods that also calls PlaySound.

```cs
public static void PlayBuzzer()
```
Calls PlaySound(`Buzzer`, 10).

```cs
public static void PlayScrollSound()
```
If `sounds[10]` isn't playing anything, PlaySound is called to play the `Audio/Sounds/Scroll` AudioClip on `sounds[10]` with a pitch of 1.0 and a volume of 0.65. If this is called while in the `pausemenu` with a `windowid` of 4 or higher TODO: why "or higher" ??? , `sounds[10]` volume gets set to `svolume` so it follows the working value in the settings.

If `sounds[10]` was playing something when this is called, this method does nothing.

```cs
public static void PlaySoundAt(string sound, float volume, Vector3 position)
```
Calls AudioSource.PlayClipAtPoint with the following parameters:

- clip: obtained by loading the `Audio/Sounds/X` AudioClip where `X` is `sound`
- position: `position`
- volume: GetSoundDistance(`position`) * `volume` * `soundvolume`

## Stopping sounds
It's possible to stop a sound after it was played if it's still playing. This is mainly useful to stop SFX that were played on loop. It involves a method called StopSound and there's 2 style of it: one that stops all `sounds` playing something and one that stops a specific `sounds` element.

```cs
public static void StopSound(AudioClip clip)
public static void StopSound(AudioClip clip, float delay)
public static void StopSound(string clipname)
public static void StopSound(string clipname, float delay)
```
This calls Stop on every `sounds` whose clip is `clip` to stop playing the AudioClip on any of these AudioSource. If `delay` is above 0.0, a FadeSound coroutine is started instead with the matching `sounds` element and the `delay` value to smoothly fade the clip.

If SoundVolume returns false (meaning sounds are muted), this method does nothing.

On overload that takes a string instead of an AudioClip, the AudioClip is obtained by loading the asset at `Audio/Sounds/X` where `X` is the string provided.

On overloads without a `delay` parameter, the default value is 0.0.

```cs
public static void StopSound(int clip)
public static void StopSound(int clip, float delay)
```
This calls Stop `sounds[clip]` to stop playing the AudioClip on the AudioSource. If `delay` is above 0.0, a FadeSound coroutine is started instead with `sounds[clip]` and the `delay` value to smoothly fade the clip.

If SoundVolume returns false (meaning sounds are muted), this method does nothing.

On the overload without a `delay` parameter, the default value is 0.1.

There's also a way to stop a sound with fading using FadeSound:

```cs
public static IEnumerator FadeSound(AudioSource sound, float speed)
```
Decreases the volume of `sound` by TieFramerate(`speed`) every frame until it reaches 0.0 or `sound` doesn't play anymore. Stop is called on `sound` once this is over.

## Sound distance
Sounds also have a concept of distance where closer sounds could sound louder than sounds playing further away. This is a concept present in [EntityControl](../Entities/EntityControl/EntityControl.md)'s `sound` AudioSource.

The volume scaling is done through a method called GetSoundDistance. There's 2 style: either using a position to determine the volume or the distance as a float.

```cs
public static float GetSoundDistance(Vector3 position)
public static float GetSoundDistance(Vector3 position, float maxdistance)
```
Returns a number between 0.0 and 1.0 where 0.0 means `position` is `maxdistance` or more units away from `MainCamera` position and 1.0 means `position` is the same as `MainCamera` position. It's a method that's meant to be used to compute the volume a sound should have relative to the `MainCamera` so closer sounds louder than sounds located further away.

The overload without a `maxdistance` parameter has its value default to 25.0.

```cs
public static float GetSoundDistance(float distance)
public static float GetSoundDistance(float distance, float maxdistance)
```
Returns a number between 0.0 and 1.0 where 0.0 means `distance` is `maxdistance` or more and 1.0 means `distance` is 0.0 or lower. It's a method that's meant to be used to compute the volume an entity should have when comparing its `camdistance` to `sounddistance` so entities located closer sounds louder than entities located further away.

The overload without a `maxdistance` parameter is UNUSED, but rmains functional and it has `maxdistance` default to 25.0.

## Other utilities methods
Here are other potentially useful methods to work with `sounds`.

```cs
public static int SoundIsPlaying(string name)
```
Returns the index of the first `sounds` that `isPlaying` and whose clip.name is `name`. If no `sounds` is found with these conditions, -1 is returned.

```cs
public static bool SoundVolume()
```
Returns true if sounds aren't muted, false otherwise. More precisely, the method returns true if any of the following conditions is true:

- `soundvolume` is above 0.0 and there's no `pausemenu` displayed
- `soundvolume` is above 0.0 and there is a `pausemenu` displayed, but its `windowid` isn't 4 (not on the settings window)
- There is a `pausemenu` displayed with a `windowid` is 4 (settings window) and `pausemenu`.`svolume` is above 0.0. This essentially means that if the game is on the PauseMenu's settings window, the working value of the sound volume setting must not be muted and it takes precedence over `soundvolume` in these conditions
