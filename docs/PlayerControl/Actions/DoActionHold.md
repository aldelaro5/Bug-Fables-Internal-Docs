TODO: detail a bit more what `digging` and `shield` does in the overworld

# DoActionHold
This is a method that is only called by [GetInput](../GetInput.md) when processing the ability input in the following circumstances:

- The input was held for 20.0 or more frames
- The input was released, but the player was `digging` and `keepdig` was above 0.0 (it's still active)

The method attempts to perform the leader party member (`playerdata[0]`)'s hold field action. However, it's possible that the conditions for the actions aren't met. If that's the case, [DoActionTap](DoActionTap.md) is called instead which treats it as a tap action instead of a hold.

Before attempting the action, `lastactionid` is always reset to 0, but it won't be set in this method further.

The specific action depends on the `playerdata[0]`'s `animid` (not its entity's [animid](../../Enums%20and%20IDs/AnimIDs.md), but they should match under normal gameplay).

## `Bee`'s hold action (Fly)
This action is only performed if all of the following conditions are fufilled:

- [flags](../../Flags%20arrays/flags.md) 19 is true (obtained Fly). Otherwise, `actionroutine` is set to a new [DoActionTap](DoActionTap.md) call if the `beemerang` is null (meaning a throw isn't already in progress)
- There are more than 1 `playerdata`
- `buttonhold` is false TODO: ???
- `canfly` is true
- `jumproutine` is null (JumpTo isn't in progress)

If all of the above conditions are fufilled, the action is performed:

- If `startheight` is null, it is initialised to the player y position
- entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) set to 102
- `flying` set to true
- entity.`overrideanim` set to true
- entity.`rigid` has its gravity disabled

The actual flying logic is maintained by LateUpdate using `startheight` to change the y position.

## `Beetle`'s hold action (Dig)
It's possible this action is called even when the player is already `digging` which would simply resume the existing action.

This action is only performed (or resumed) if all of the following conditions are fufilled:

- [flags](../../Flags%20arrays/flags.md) 18 is true (obtained Dig). Otherwise, `actionroutine` is set to a new [DoActionTap](DoActionTap.md) call
- `candig` is true
- The entity is `onground`
- A Raycast from the player position + 0.1 in y heading down in y for up to 0.25 distance registers a hit with anything on layer 8 (`Ground`)

If any of the conditions above aren't fufilled with the exception of the flag (since it causes the tap action to happen), [CancelAction](CancelAction.md) is called instead which stops any existing `digging`.

If all of the above conditions are fufilled, the action is performed (or resumed):

- If not `digging` already and entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) isn't 101, the `Dig` sound plays at 0.7 volume
- entity.`animstate` set to 101
- entity.`overrideanim` set to true
- entity's y `spin` set to 25.0
- `startdig` set to true (this eventually will cause FixedUpdate to set `digging` to true)

This is where the logic differs between a new action and a resume which is checked by the `digging` value as if it's false, it means this is a new digging action and if it's true, it's resuming an existing digging action.

### New action

- entity.[StopMoving](../../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) with targetstate being the entity.`animstate`
- If `diggingpart` is null, it is initialised to a new `DirtFlying` particles at the player position with -90.0 x angle and -1.0 alivetime (infinite)

### Resume an existing action

- `diggingpart` is moved offscreen then destroyed in 1.0 second
- If `tunnelpart` is null:
    - RefreshPlayer with onlycollider is called which will cause [ForceHitWall](../../Entities/EntityControl/EntityControl%20Methods.md#forcehitwall) to be called on the player entity if not `inevent` and not in a `battle`
    - `tunnelpart` is initialed to a new `Digging` particles childed to the player at the player position with -90.0 x angle, -1.0 alivetime (infinite) and 2760 rendersort

## `Moth`'s hold action (Shield)
It's possible this action is called even when the player is already in a `shield` which would simply resume the existing action.

This action is only performed (or resumed) if [flags](../../Flags%20arrays/flags.md) 20 is true (obtained Shield). Otherwise, `actionroutine` is set to a new [DoActionTap](DoActionTap.md) call

Here's the logic of the action if it is performed (or resumed):

- entity.`overrideanim` is set to true
- entity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to a value that depends on `delta`'s magnitude:
    - \>= 0.1: 123
    - \< 0.1: 122
- If the player isn't already in a `shield` (it's a new action):
    - TeleportFollowers is called
    - `Shield` sound plays on the entity
- `shield` is set to true
