# ChaseWhenAnim
Perform the same logic than [ChasePlayer](ChasePlayer.md) when the entity.[animstate](../../EntityControl/Animations/animstate.md) is `Chase`. Otherwise, if the entity.`animstate` isn't the frequency, entity.`animstate` is set to it with a 20.0 frames `behaviorcooldown` (prevents a [WalkWhenAnim](WalkWhenAnim.md) doing a [Wander](Wander.md) DoBehavior cycle to process for 20.0 frames when entity.`animstate` is `Idle`) and a [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) call on the entity.

## Frequency meaning
An [animstate](../../EntityControl/Animations/animstate.md) to set the entity.`animstate` when the current one is different and it's not `Chase`.

## About this behavior and animation events
This behavior is intended to be used with an [animstate](../../EntityControl/Animations/animstate.md) as the frequency whose clip contains an animation event that sets the entity.`animstate` to `Chase`. This is because under normal circumstances, there is no logic to change the entity.`animstate` to `Chase` offered by this behavior. It is intended to be used whenever the timing of the [ChasePlayer](ChasePlayer.md) needs to be carefully determined by the animation clip instead of immediately triggering when the player gets `inrange`.

It can be used as the `inrange` behavior while [WalkWhenAnim](WalkWhenAnim.md) is the default one. This allows `behaviorcooldown` to take effect and not change the entity.`animstate` too frequently.

## DoBehavior
If the entity.`animstate` is `Chase`, this acts as an alias to [ChasePlayer](ChasePlayer.md).

Otherwise, if the current entity.`animstate` isn't the `frequency`:

- The `behaviorcooldown` is set to 20.0 (prevents a [WalkWhenAnim](WalkWhenAnim.md) doing a [Wander](Wander.md) DoBehavior cycle to process for 20.0 frames when entity.`animstate` is `Idle`)
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- The entity.`animstate` is set to frequency
