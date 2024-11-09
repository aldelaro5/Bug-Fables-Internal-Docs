# Movement
This is a method that is called on every LateUpdate where the player isn't `dashing` (if they were, the movement logic would be overriden by [DashBehavior](DashBehavior.md)).

It handles movement logic using the information gathered from Update's movement inputs processing, but it only does if all of the following conditions are true:

- We aren't in an instance.`pause`
- We aren't in an instance.`minipause`
- We aren't in instance.`inevent`
- The [message](../SetText/Notable%20states.md#message) lock is released
- entity.`rigid` exists

There are 2 possible logic:

- If the -2 input (any movement input) is held, movement logic happens
- Otherwise (no movement input are held), the stop logic happens

## Movement logic
This only happens when any movement inputs is held:

- If entity.`hitwall` is false:
    - The `spd` value is determined and their x/z components will be set to the entity.`rigid` x/z velocity (the y velocity is unchanged). The first cases below that applies is selected and its matching `spd` value will be used:
        - The player is `flying`: `spd` is `lastdelta` * entity.`speed` / 2.0
        - The player is `digging` or is in a `shield`: `spd` is `lastdelta` * entity.`speed` / 1.5
        - MainManager.`analog` is 0: `spd` is `lastdelta` * entity.`speed`
        - None of the above cases applies: `spd` is `walkdelta`
    - If the player isn't `flying`, entity.[animstate](../Entities/EntityControl/Animations/animstate.md) is set to 1 (`Walk`)
- Otherwise:
    - [StopMoving](../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) is called on the entity with a targetstate of 1 (`Walk`)
- If the down input is held:
    - instance.`camspeed` is set to a lerp from the existing one to 0.25 with a factor of TieFramerate(0.01)
    - If entity.`hitwall` is false:
        - instance.`camoffset2` is set to a lerp from the existing one to -`CamDir`.forward.normalized * 1.25 with a factor of TieFramerate(0.025)
    - Otherwise (entity.`hitwall` is true):
        - ReturnOffset called which sets instance.`camoffset2` to a lerp from the existing one to Vector3.zero with a factor of TieFramerate(0.01)
- Otherwise (the down input isn't held):
    - If instance.`changecamspeed` is false, instance.`camspeed` is set to 0.1
    - ReturnOffset called which sets instance.`camoffset2` to a lerp from the existing one to Vector3.zero with a factor of TieFramerate(0.01)

What happens next relate to the movement sound and it depends if the player is in a `submarine` or not. Only one of the 2 cases applies.

### `submarine` sonar sound
It only applies if `digging` is true (the player is underwater) and that `Sonar` isn't playing on entity.`sound`. If these conditions are fufilled, `Sonar` is played on entity.`sound` without loop at 0.75 volume.

### No `submarine` footstep sound
First, entity.`sound` is set to not loop.

From there, the footstep sound logic only applies if all of the following conditions are fufilled:

- entity is `onground`
- The player isn't `digging`
- entity.`sound` isn't playing

If all of the above are fufilled, `footstep` is decreased by `framestep`.

From there, if `footstep` reached 0.0 or below, the `Footstep` sound plays on the entity with `footstep` being set to 7.5. This means that as long as the player keeps moving, there will be a sound every 7.5 frames of movement. This resets when movement is stopped as explained in the stopping logic below.

## Stop logic
This only happens when no movement input is held:

- `footstep` is reset to 0.0 (reset the timer for the sound when moving)
- [ForceHitWall](../Entities/EntityControl/EntityControl%20Methods.md#forcehitwall) called on the entity
- Movement is stopped with a method that depends on if the player is in a `submarine`:
    - If it is, [StopMoving](../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) is called on the entity with a targetstate of 0 (`Idle`)
    - If it's not, entity.`rigid` velocity is lerped from the existing on to Vector3.zero with a factor of TieFramerate(0.025)
- ReturnOffset called which sets instance.`camoffset2` to a lerp from the existing one to Vector3.zero with a factor of TieFramerate(0.01)
