# ShootProjectile
Shoot a projectile ???

## Frequency meaning
None ???

## DoBehavior
First, if `returntoheight` is true, the entity `height` is set to a lerp from the existing one to its `initialheight` with a factor of 0.1.

Then, if the entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity without smoothing and without changing the [animstate](../../EntityControl/Animations/animstate.md).

Finally, a ShootProjectile coroutine is started and set to `behaviorroutine`.

## ShootProjectile
TODO