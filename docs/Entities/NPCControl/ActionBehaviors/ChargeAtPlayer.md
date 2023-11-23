# ChargeAtPlayer
Charges at the player with `attacking` set to true during the movement towards the player (this will allow an [Enemy](../NPCType.md#enemy) to initiate a battle with the advantage if it collides with the player and all conditions are met to start a battle with the advantage).

## Frequency meaning
None.

## DoBehavior
If `returntoheight` is true, the entity.`height` is set to a lerp from the existing one to entity.`initialheight` with a factor of 0.1.

Finally, a ChargeAtPlayer coroutine is started and set to `behaviorroutine`.

## ChargeAtPlayer
This is a private coroutine specific to this behavior. It receives the behavior type (which is always this type) and a cooldown time (the `actioncooldown`) in frames that would have been used to smoothly return the entity.`height` to entity.`initialheight`, but in practice, it's always 0.0 which disables this feature. The coroutine call is assigned to `behaviorroutine` which allows the NPCControl to stop it if needed.

- entity.`rigid` is unlocked with [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid)
- The `Find` sound is played on the entity
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to `Chase` unless entity.`height` is higher than 0.1 where the value is set to `AirTackle`
- If the entity is `onground`, [Jump](../../EntityControl/EntityControl%20Methods.md#jump) is called on it
- entity.`emoticonid` is set to 2 (the red !)
- entity.`emoticoncolldown` is set to 300.0
- 0.25 seconds are yielded
- entity.`overrideonlyflip` is set to false
- `attacking` is set to true
- [MoveTowards](../../EntityControl/EntityControl%20Methods.md#movetowards) is called on the entity with its `speedmultiplier`, new animstate set earlier for the walkstate, `Idle` for the stopstate ignoring the y component. The position is the player's + the direction from the player to this NPCControl normalised * 1.5. That position is however limited to a radius whose center is the entity. `startpos` and the radius being `radiuslimit`. Essentially, this moves the NPCControl to slightly undeshoot the player when going towards it
- entity.`detect` is ensured to be created
- [DetectDirection](../../EntityControl/EntityControl%20Methods.md#detectdirection) is called on the entity using the player position
- As long as the entity.`forcemove` is ongoing and the entity.`hitwall` is false:
  - entity.`emoticonid` is set to 2 (the red !)
  - entity.`emoticoncolldown` is set to 5.0
  - If entity.`initialheight` is above 0.1, entity.`height` is set to a lerp from entity.`initialheight` to 0.25 with a factor of (the x/z square distance between this NPCControl and entity.`forcetarget`) / (the x/z square distance between entity.`startpos` and entity.`forcetarget`). NOTE: the math is likely incorrect, but the main point here is the NPCControl lowers faster as the player is further away from it and slows down as it gets closer
  - A frame is yieled
- A frame is yieled
- entity.`sprite` angles are set to the return of a FlipAngle call on the entity
- `attacking` is set to false
- entity.`overrideonlyflip` is set to false
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity with the `Idle` targetstate
- If the entity is `onground`, [Jump](../../EntityControl/EntityControl%20Methods.md#jump) is called on it
- 0.2 seconds are yieled
- All frames are yieled until entity.`onground` goes to true
- A frame is yielded
- Some dead code is present here where the entity.`height` would be lerped towards entity.`initialheight`, but this is never applicable in practice because the passed cooldown is `actioncooldown` which is always 0.0 here
- A frame is yielded
- entity.`height` is set to entity.`initialheight`
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
