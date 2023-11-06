# StealthAI
???

## Frequency meaning
The same than [SetPath](SetPath.md), but with the addition that if it's 5555, the [animstate](../../EntityControl/Animations/animstate.md) is set to `Sleep` just before the `forcemove` check.

## SetUp
The entity's `detect` is ensured to be created which makes sure the entity has a wall detector. From there, a new StealthCheck component gets childed to the `detect` with a local position of Vector3.zero and a `parent` field of this NPC. Finally, the entity's `alwaysactive` and `extratimer` are set to true.

## DoBehavior
The same than [SetPath](SetPath.md), but with the addition the `frequency` field has as mentioned above.

TODO: clearly not the full story ???

## Update (Inactive, every 3 frames)
Normally, when an entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove), [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on it, but this behavior is an exception to this where it will not be called here.

## Update (Common, end)
As long as the behavior exist on the NPCControl, `actioncooldown` is set to 1.0 if all the following conditions are true if the player is present and not digging while the [message](../../SetText/Notable%20states.md#message) lock is active.

## LateUpdate (RefreshPlayer)
After the new `inrange` value is set, and the new value is true a [StealthSpot](ActionBehaviors/StealthAI.md#stealthspot) coroutine is called. There is an exception where this doesn't happen if this is an `Enemy` when the `freezecooldown` or `dizzytime` hasn't expired yet.

## OnTriggerEnter (`NPC` only)
A StealthSpot coroutine starts if the other collider is the `beemerang` and the square distance between this `NPC` and the player is less than 30.0

## StealthSpot
TODO

## OnCollisionEnter
If it's an `NPC`, the other gameObject tag isn't `Hornable` and the other's Hornable type is `IceCube`, [ShatterDroppletIce](../ShatterDroppletIce.md) is called on the other's Hornable `parent`.