# StealthAI
Gives the ability of the NPCControl to spot the player if it enters its field of vision (represented by 2 trigger colliders) which has the effect of triggering an [event](../../../Enums%20and%20IDs/Events.md). The event id and the wideness of the field of vision are configurable as well as an option to set the entity.[animstate](../../EntityControl/Animations/animstate.md) to `Sleep` at the start of the DoBehavior cycle.

## Frequency meaning
The same than [SetPath](SetPath.md), but with the addition that if it's 5555, the entity.[animstate](../../EntityControl/Animations/animstate.md) is set to `Sleep` at the start of the DoBehavior cycle (right after the `returntoheight` logic).

## Additional data
- `vectordata`: The same than [SetPath](SetPath.md)
- `battleids[0]`: An [event](../../../Enums%20and%20IDs/Events.md) id that will be started when the player is spotted
- `battleids[1]`: A value that represents how wide the NPCControl's vision is. It's the z size of the trigger BoxCollider of the StealthCheck, a component attached to the entity.`detect` that can spot the player if it collides with it and its SphereCollider. It also influences the z center of the BoxCollider with the value being half of this one floored.

## SetUp
- entity.`detect` is ensured to be created which makes sure the entity has a wall detector
- A new StealthCheck component gets childed to the entity.`detect` with a local position of Vector3.zero and a `parent` field of this NPCControl. 
- entity.`alwaysactive` is set to true
- entity.`extratimer` is set to true

## DoBehavior
The same than [SetPath](SetPath.md), but with the addition that if the frequency is 5555, the entity.[animstate](../../EntityControl/Animations/animstate.md) is set to `Sleep` at the start of the DoBehavior cycle (right after the `returntoheight` logic).

## Update (Inactive, every 3 frames)
Normally, when an entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on it, but this behavior is an exception to this where it will not be called here.

## Update (Common, end)
As long as the behavior exist on the NPCControl, `actioncooldown` is set to 1.0 if the player is present and not `digging` while the [message](../../../SetText/Notable%20states.md#message) lock is active. This has the overall effect to have the underlying [SetPath](SetPath.md) logic operate as if it has a frequency of 1.0 meaning 1.0 frames of cooldown between movement.

## LateUpdate (RefreshPlayer)
After the new `inrange` value is set and the new value is true a StealthSpot coroutine is called. There is an exception where this doesn't happen if this is an [Enemy](../NPCType.md#enemy) when the `freezecooldown` or `dizzytime` hasn't expired yet.

## OnTriggerEnter (If this is an [NPC](../NPCType.md#npc))
A StealthSpot coroutine starts if the other collider is the player.`beemerang` and the square distance between this NPCControl and the player is less than 30.0

## OnCollisionEnter (If this is an [NPC](../NPCType.md#npc))
If the other gameObject tag is `Hornable` and the other's Hornable type is `IceCube` (meaning it's a [Dropplet](../ObjectTypes/Dropplet.md) ice cube), [ShatterDroppletIce](../ObjectTypes/Dropplet.md#shatterdroppletice) is called on the other's Hornable `parent` (the `Dropplet` NPCControl). This basically shatters the cube whenever an NPCControl with this behavior present (`inrange` or not) collides with it.

## StealthSpot
This is a public coroutine specific to this behavior (the only reason it's public is for StealthCheck to be able to start it). Its job is to verify that the NPCControl has indeed spotted the player and to process it if it has.

A Linecast will be performed from this NPCControl position + Vector3.up to the player position + Vector3.up for only layers `Ground`, `NoDigGround` and `Player`.

For the spotting to register, all of the following conditions must be true:

- `startlife` is above 20.0
- The Linecast hit any `playerdata` entity
- The player is free (ignoring flying)
- The player isn't `digging`
- `battleids[0]` isn't negative

If the above are met, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity followed by the [event](../../../Enums%20and%20IDs/Events.md) whose id is `battleids[0]` starting with this NPCControl as the caller.

No matter if the spotting was processed or not, a frame is yielded.

## StealthCheck
This is a component specifically involved with this behavior as it is attached to the entity.`detect` on SetUp. Its job is to be the way the NPCControl can spot the player with 2 trigger colliders.

These colliders are added on the component's Start:

- A trigger SphereCollider with radius 1.5 and default position (meaning at the entity.`detect`'s position)
- A trigger BoxCollider with size (2.5, 2.0, NPCControl's `battleids[1]`) and center (0.0, 1.5, half of NPCControl's `battleids[1]` floored)

The Start also puts the entity.`detect` GameObject in layer 2, the built in layer of Unity to ignore raycasts.

From there, the only interesting logic this component has is an OnTriggerEnter where the method will call StealthSpot under the following conditions:

- The player must be present and free (flying counts as not free)
- If the other collider is the player.`beemerang`, it must be less than 30.0 away from the entity.`detect`
- If it's the player, the internal cooldown of the component must be expired

If the above is met, SteatlhStop is called which will verify that it is actually a spot. It also will set the internal cooldown to 3.0.

What this cooldown does is it prevents the player collider from doing anything for a certain time in frames. This cooldown is decremented on LateUpdate if it hasn't expired yet. The overall effect is no player collider spotting can happen if StealthSpot was called up to 3 frames ago by this component.