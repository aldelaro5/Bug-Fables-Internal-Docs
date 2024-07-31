# WalkWhenAnim
Perform [Wander](Wander.md) when the entity [animstate](../../EntityControl/Animations/animstate.md) is `Walk` (or `Idle` if the `behaviorcooldown` expired) or change it to the frequency when it's not with a `behaviorcooldown` of 20.0 frames (prevents another [Wander](Wander.md) cycle to process for 20.0 frames when entity.`animstate` is `Idle`).

## Frequency meaning
An [animstate](../../EntityControl/Animations/animstate.md) to set the entity one when the current one is different and it's not `Walk` (or `Idle` when the `behaviorcooldown` expired).

## About this behavior and animation events
This behavior is intended to be used with an [animstate](../../EntityControl/Animations/animstate.md) as the frequency whose clip contains an animation event that sets the entity.`animstate` to `Walk` (or after, but only applicable when `behaviorcooldown` expires). This is because under normal circumstances, there is no logic to change the entity.`animstate` to these 2 that is offered by this behavior. It is intended to be used whenever the timing of the [Wander](Wander.md) needs to be carefully determined by the animation clip instead of immediately triggering when the behavior gets switched to.

It can be used as the default behavior while [ChaseWhenAnim](ChaseWhenAnim.md) is the `inrange` one. This allows `behaviorcooldown` to take effect and not change the entity.`animstate` too frequently.

## DoBehavior
If the entity.`animstate` is `Walk` (or `Idle` if the `behaviorcooldown` expired), this acts as an alias to [Wander](Wander.md).

Otherwise, if the current entity.`animstate` isn't the frequency:

- The `behaviorcooldown` is set to 20.0 (prevents another [Wander](Wander.md) cycle to process for 20.0 frames when entity.`animstate` is `Idle`)
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- The entity.`animstate` is set to frequency
