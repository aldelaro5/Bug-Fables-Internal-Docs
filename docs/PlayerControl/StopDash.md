# StopDash
This coroutine allows to stop a [DashBehavior](DashBehavior.md) and must be called to stop it when the player is `dashing`. [CancelAction](Actions/CancelAction.md) can call it automatically in these circumstances.

```cs
public IEnumerator StopDash(bool wall)
```

## Parameters

- `wall`: Tells if the stopping is due to hitting a wall which implicates more visual effects such as a `Death3` sound

## Procedure

- If wall is true:
    - `Death3` sound plays
    - entity.`forcemove` is set to false
    - entity.[animstate](../Entities/EntityControl/Animations/animstate.md) set to 11 (`Hurt`)
    - ShakeScreen called with an ammount of (0.1, 0.1, 0.1) for 0.2 time
- `smoke` is moved offscreen then gets destroyed in 1.5 seconds
- If `tbox` exists, it is destroyed
- entity.`detect` tag is reset to `Untagged`
- Yield for a frame
- entity.`rigid` x and z velocity are reset to 0.0
- `diggingpart` reset to null
- `dashing` is set to false
- Yield for 0.25 seconds
- entity.`animstate` set to 0 (`Idle`)
- entity.`jumpcooldown` set to 5.0 so no jumps can happen for the next 5.0 frames
- `actioncooldown` set to 5.0 so no [DoActionTap](Actions/DoActionTap.md) can be done for the next 5.0 frames
- `movecd` reset to 0.0
- [CancelAction](Actions/CancelAction.md) is called if all of the following conditions are true:
    - We aren't in an instance.`pause`
    - We aren't in an instance.`minipause`
    - We aren't in instance.`inevent`
    - The [message](../SetText/Notable%20states.md#message) lock is released
    - The player isn't `digging`
    - The player isn't `flying`
