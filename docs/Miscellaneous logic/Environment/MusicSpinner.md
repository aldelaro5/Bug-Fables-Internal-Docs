# MusicSpinner
A component meant to be attached on any GameObject via Unity, it will get a RigidBody component on its own on Start.

This component is only used in the `BugariaTheater` map to implement GameObjects spinning as a sound is played multiple times in succession with varying pitch and length as it is spinned using `Beetle`'s Horn Slash field ability. When it has spun enough, an item is launched from it if it wasn't obtained already.

While it's only used there, it has several configurations available. Here are the public fields:

- `notes`: The list of pitches and length of each times to play the `soundclip` during a full spinning cycle. Each x componets represents the timestamps the playback should happen in seconds and each y components represents the pitch to use for the playback
- `maxtime`: The maximum playback time of all the `notes` in a full spinning cycle before restarting from the first `notes` element
- `spinlimit`: The spin value requited (the value increases in increments of `spinhit` each Horn Slash) before the item spawns if it wasn't obtained already
- `maxspin`: The maximum spin value (the value increases in increments of `spinhit` each Horn Slash)
- `spinstop`: An amount of frames that presents the value to decrease the spin value on each LateUpdate (the value increases in increments of `spinhit` each [Horn Slash](../../PlayerControl/Field%20abilities.md#horn-slash))
- `spinhit`: The increment to increase the spin value of the `spinner` on each Horn Slash (done on OnTriggerEnter with any `BeetleHorn` tagged colliders)
- `musiclower`: The volume multiplier to use as the destination of a lerp applied to `music[0]`.volume on each LateUpdate while the music is playing (the lerp is done from the existing volume with a factor of the spin / `maxspin` where spin is clamped from 0.0 to `maxspin` so it approaches the scaled volume more as the cylinder spins more). This is mostly used to fade the current music using a value between 0.0 and 1.0
- `flag`: A [flag](../../Flags%20arrays/flags.md) slot indicating if the item was obtained already (also bound to the [NPCControl](../../Entities/NPCControl/NPCControl.md)'s `activationflag` of the created item when launched if `itemtype` isn't 3 so not a Crystal Berry). If `itemtype` is 3 (Crystal Berry), this value is a [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) instead
- `itemtype`: The type of item launched from the cylinder:
    - 0: Standard [item](../../Enums%20and%20IDs/Items.md)
    - 1: Key item
    - 2: [Medal](../../Enums%20and%20IDs/Medal.md)
    - 3: Crystal Berry (with the crystalbflags specified in `flag`)
- `itemid`: The item, medal id or crystalbflags of the launched item. NOTE: If `itemtype` is 3 (Crystal Berry), this value MUST match the `flag` because they're both used differently, but represents the same thing
- `itemtime`: The amount of time in frames to spawn the item before it disappears. If it's -1.0, the item won't disappear
- `itempos`: The position to spawn the item before it is launched
- `bouncepos`: The direction to launch the item once it is spawned
- `spinner`: The list of GameObject to rotate using the spin value. All of them's isStatic will be set to false if they weren't when they spin
- `rotation`: The axis of rotate of each `spinner` as they rotate
- `soundclip`: The base sound to use to play the `notes`
- `musicvolume`: This field is UNUSED and it doesn't do anything.

On Start, the GameObject gets a Rigidbody component added without gravity in kinematic. Additionally, if the item hasn't been obtained yet and the `Detector` medal is equipped, MainManager.`map`.`hiddenitem` is set to 100 which lets the beeper sound play.

All the spinning and item spawning logic happens in LateUpdate and the logic to detect Horn Slash to start the spin happens in OnTriggerEnter where the collider has the `BeetleHorn` tag.
