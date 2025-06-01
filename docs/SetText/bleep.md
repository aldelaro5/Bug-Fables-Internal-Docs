# Bleep
SetText has a feature exclusive to [dialogue mode](Dialogue%20mode.md) called bleep which are special dialogue SFX composed of a very short AudioClip with a defined pitch that plays for around half the letters rendered to simulate the character talking sound. The particular one played is dependant on the current `tailtarget` (the speaker) and it more specifically depends on their animid data. Check the animid entity data documentation to learn more.

The way bleeps work is they have a dedicated AudioSource specifically to play them which is the `bleeps` field. Only bleeps are played there, nothing else.

The bleeps themselves are located in the `Resources/audio/sounds/dialogue` asset directory. While there are only about a dozen there, most of their diversity comes from tuning a pitch value.

While SetText has a command to play a bleep called [bleep](Individual%20commands/Bleep.md), it's possible to operate on the system manually using 2 methods: GetBleep and PlayBleep.

## GetBleep

```cs
public static string GetBleep(EntityControl entity)
public static string GetBleep(EntityControl entity, float volume)
```
Returns a string that forms a valid bleep SetText command with the following parameters:

- sound: `entity`.`dialoguebleepid`
- pitch: `entity`.`bleeppitch`
- volume: `volume`

## PlayBleep

```cs
public static void PlayBleep(AudioClip bleep, float bleeppitch, float bleepvol, int i)
```
Play the `bleep` clip on `bleeps` if i is an even number and `bleeps` wasn't playing already. The pitch used will be `bleeppitch` + a random number between -0.05 and 0.05 and the volume will be `bleepvol` * `bleepvolume` (or `pausemenu`.`dvolume` if the game is in a `pausemenu` so it uses the working value in the settings).

The `i` parameter is meant to be a number that changes each calls to this. This is more suited for SetText where it sends the current character index being processed in the char loop so the playback happens around half the time a character is rendered.
