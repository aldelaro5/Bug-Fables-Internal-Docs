# ChaseWhenAnim
Perform [ChasePlayer](ChasePlayer.md) when the entity [animstate](../../EntityControl/Animations/animstate.md) is `Chase` or change it to the frequency when it's not.

## Frequency meaning
An [animstate](../../EntityControl/Animations/animstate.md) to set the entity one when the current one is different and it's not `Chase`.

## DoBehavior
If the entity `animstate` is `Chase`, this acts as an alias to [ChasePlayer](ChasePlayer.md).

Otherwise, if the current `animstate` isn't the `frequency`:
- The `behaviorcooldown` is set to 20.0
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on the entity
- The `animstate` is set to frequency