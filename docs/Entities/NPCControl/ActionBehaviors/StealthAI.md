# StealthAI
???

## Frequency meaning
The same than [SetPath](SetPath.md), but with the addition that if it's 5555, the [animstate](../../EntityControl/Animations/animstate.md) is set to `Sleep` at the start of the DoBehavior cycle (right after the `returntoheight` logic).

## SetUp
- entity.`detect` is ensured to be created which makes sure the entity has a wall detector
- A new StealthCheck component gets childed to the `detect` with a local position of Vector3.zero and a `parent` field of this NPCControl. 
- entity.`alwaysactive` is set to true
- entity.`extratimer` is set to true

## DoBehavior
The same than [SetPath](SetPath.md), but with the addition that if the frequency is 5555, the entity.[animstate](../../EntityControl/Animations/animstate.md) is set to `Sleep` at the start of the DoBehavior cycle (right after the `returntoheight` logic).

## Update (Inactive, every 3 frames)
Normally, when an entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove), [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called on it, but this behavior is an exception to this where it will not be called here.

## Update (Common, end)
As long as the behavior exist on the NPCControl, `actioncooldown` is set to 1.0 if the player is present and not `digging` while the [message](../../SetText/Notable%20states.md#message) lock is active.

## LateUpdate (RefreshPlayer)
After the new `inrange` value is set, and the new value is true a [StealthSpot](ActionBehaviors/StealthAI.md#stealthspot) coroutine is called. There is an exception where this doesn't happen if this is an [Enemy](../NPCType.md#enemy) when the `freezecooldown` or `dizzytime` hasn't expired yet.

## OnTriggerEnter (If this is an [NPC](../NPCType.md#npc))
A StealthSpot coroutine starts if the other collider is the player.`beemerang` and the square distance between this NPCControl and the player is less than 30.0

## OnCollisionEnter (If this is an [NPC](../NPCType.md#npc))
If the other gameObject tag is `Hornable` and the other's Hornable type is `IceCube` (meaning it's a [Dropplet](../ObjectTypes/Dropplet.md) ice cube), [ShatterDroppletIce](../ObjectTypes/Dropplet.md#shatterdroppletice) is called on the other's Hornable `parent` (the `Dropplet` NPCControl). This basically shatters the cube whenever an NPCControl with this behavior present (`inrange` or not) collides with it.

## StealthSpot
TODO

## StealthCheck
TODO
