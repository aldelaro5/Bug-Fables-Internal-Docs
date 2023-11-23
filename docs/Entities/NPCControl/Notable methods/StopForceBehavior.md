# StopForceBehavior
This is a public method that aims to cleanly stop all forced behaviors or a behavior that uses a coroutine. Nothing happens if it's a `dummy`.

- `forcebehavior` is set to null
- `behaviorroutine` is stopped then set to null
- If we aren't `inevent`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- entity.`overrideonlyflip` is set to false if it was true
- `attacking` is set to false
- If it's an [Enemey](../NPCType.md#enemy) and we aren't `inevent`, the entity.`sprite` angles are set to (0.0, the y FlipAngle of the entity,  0.0)