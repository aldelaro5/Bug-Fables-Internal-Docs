# CancelAction
This is a method that can cancel any ongoing [DoActionTap](DoActionTap.md) and [DoActionHold](DoActionHold.md). by cleaning up everything they were doing. There are 2 overloads available:

This one ends up calling the one below where keepbeerang is false

```cs
public void CancelAction()
```

This is the main overload where the cancellation logic is

```cs
public void CancelAction(bool keepbeerang)
```

## Parameters

- `keepbeerang`: If false, the `beemerang` will get destroyed if it exists (this is only called with true when colliding with a [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md))

## Procedure

- `movecd` is reset to 0.0 so any existing movement's frame counter resets
- If the player is `dashing`, [StopDash](../StopDash.md) is called without wall
- entity gets [StopForceMove](../../Entities/EntityControl/EntityControl%20Methods.md#stopforcemove) called
- The `tbox` is destroyed if it exists
- If the player is `digging`, `DigPop2` sound plays at 0.7 volume
- `flycooldown` is reset to 240.0
- `shield` is set to false
- If the player is `flying` or `digging`:
    - The entity.`sound` stops playing
    - TeleportFollowers is called
    - If there are more than 1 `playerdata`, all of them gets the following adjustement on their entity:
        - [LockRigid(false)](../../Entities/EntityControl/EntityControl%20Methods.md#lockrigid) called to unlock their `rigid`
        - `overrideanim` set to false
        - `onground` set to false (forces an animation update)
        - `spin` zeroed out
        - [animstate](../../Entities/EntityControl/Animations/animstate.md) set to 0 (`Idle`)
- `digging` is set to false
- `flying` is set to false
- `startdig` is set to false
- `diggingpart` is moved offscreen then destroyed in 1.0 seconds
- `tunnelpart` is moved offscreen then destroyed in 1.0 seconds
- `lockkeys` is set to false which unlocks most input processing
- entity.`sound` stops playing
- entity.`overrideanim` is reset to false
- If the player isn't in a `submarine`, entity.`overrideflip` is reset to false
- entity.`overridejump` is reset to false
- entity.`sound` stops playing again
- `startheight` is reset to null
- entity.`spin` is zeroed out
- entity.`rigid`'s gravity gets enabled
- If `actionroutine` is in progress, it is stopped
- If the `beemerang` exists and keebeerang is false, it is destroyed
- If `tempcamoffset` isn't null (never happens under normal gameplay since this field is UNUSED), instance.`camoffset` is set to the value
- If `icecle` exists:
    - A `Prefabs/Particles/IceShatter` GameObject is instantiated rooted at the `icecle` with -90.0 x angle, but destroyed in 2.0 seconds
    - `icecle` is destroyed
- entity.`sound` stops looping
- `tempcamoffset` is reset to null (it's not normally used)
- `action` is set to false
- `pausecooldown` is set to 10.0 so pausing isn't allowed for the next 10.0 frames
- entity.`onground` is set to false which forces an animation update
- ActionCooldown is called which sets `actioncooldown` to a value depending on `lastactionid`:
    - 1 (`Beetle`): 5.0 frames
    - Any other values (`Bee` or `Moth`): 17.0 frames
