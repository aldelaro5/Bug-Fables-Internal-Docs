# Field abilities
PlayerControl offers multiple player abilities and it manages 6 of the 7 available in the game (Beemerang Halt is managed separately in the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) logic). This page focuses more on describing what these abilities do in detail rather than their actuation or requirements to use them which are explored in [DoActionTap](Actions/DoActionTap.md) and [DoActionHold](Actions/DoActionHold.md).

This page will detail what each abilities does once actuated.

## `Bee`'s field abilities
This section will detail `Bee`s field abilities. Exceptionally, `Bee` only has 2 instead abilities because while Beemerang toss is managed in PlayerControl, Beemerang halt is managed in [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md).

### Beemerang Toss
This ability causes a [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) to be created and assigned to the player's `beemerang`. The `beemerang` is only destroyed and set to null when its travel is over or when some other circumstances cancels its travel so it is one of the only dynamically created NPCControl in the game.

The travel and colliders management logic as well as Beemerang halt is handled in NPCControl, but the game still has special logic when it comes to things that collides with the player's `beemerang` which also has a unique `BeeRang` tag. Here's what this collider allows:

- When the `beemerang` collides with an [Item](../Entities/NPCControl/ObjectTypes/Item.md), as long as some other specific conditions are fufilled, the item's `beerang` will be set to the player's `beemerang` which will have the item move alongside the beemerang and prevent any Hazard from respawning the item
- When the `beemerang` collides with an [Enemy](../Entities/NPCControl/Enemy.md), it places them in a dizzy state
- All collisions with Hazards and the player's `beemerang` are ignored
- If thrown near an [NPC](../Entities/NPCControl/NPC.md) with a [SteathAI](../Entities/NPCControl/ActionBehaviors/StealthAI.md) behavior, the player gets spotted
- During a WackaWorm game, a worm colliding with the player's `beemerang` causes them to get hit and increment the score. Also, the game's `timer` doesn't decrease while the player's `beemerang` exists
- The player's `beemerang` is the only practical way to actuate a [ScrewSwitch](../Entities/NPCControl/ObjectTypes/ScrewSwitch.md) in a sustainable fashion
- Collisions with the player's `beemerang` and these NPCControl are recognised:
    - [SavePoint](../Entities/NPCControl/ObjectTypes/SavePoint.md): Allows to actuate the save point (position needs to be close enough for [Interact(save)](../Entities/NPCControl/Notable%20methods/Interact.md) to be called)
    - [CoiledObject](../Entities/NPCControl/ObjectTypes/CoiledObject.md): Allows to untrap the object that's `trapped` in it
    - [Switch](../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../Entities/NPCControl/ObjectTypes/WaterSwitch.md): Allows to actuate all of these switches

### Bee fly
This ability's main feature is setting `flying` to true which mainly causes changes in the movement logic as well as some other visual logic in [EntityControl](../Entities/EntityControl/EntityControl.md) for up to 240.0 frames:

- entity.`rigid` no longer has gravity
- entity.`rigid` y velocity is zeroed out on each LateUpdate so only x/z movements are possible
- The player y position constantly increases up to + 1.0 every LateUpdate using a lerp from the existing one to the starting value when the flight was started (`startheight`) + 1.0 with a factor of 0.05
- The `spd` movement speed used by [Movement](Movement.md) is halved from regular movement logic

## `Beetle` abilities
This section will detail `Beetle`s field abilities. They have 4 abilities available.

### Horn slash
This ability creates a `BeetleHorn` tagged trigger collider that lives for only 0.15 seconds, but it has special properties because it is recognised by many other colliders in the game. Here is how these colliders reacts when colliding with a `BeetleHorn` tagged collider:

- [Hornable](../../Entities/NPCControl/ObjectTypes/Dropplet.md#hornable): Allows to move the object
- MusicSpinner: Allows to hit and roll the spinner
- [BreakableRock](../../Entities/NPCControl/ObjectTypes/BreakableRock.md): Allows to shake, but not destroy the rock
- [SavePoint](../../Entities/NPCControl/ObjectTypes/SavePoint.md): Allows to interact with the save to save the game
- [PushRock](../../Entities/NPCControl/ObjectTypes/PushRock.md): Allows to move the object
- [CoiledObject](../../Entities/NPCControl/ObjectTypes/CoiledObject.md): Allows to uncoil the object trapped
- [Switch](../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../Entities/NPCControl/ObjectTypes/WaterSwitch.md): Allows to actuate all of these switches
- [BeetleGrass](../../Entities/NPCControl/ObjectTypes/BeetleGrass.md): Allows to cut the grass
- [ScrewSwitch](../../Entities/NPCControl/ObjectTypes/ScrewSwitch.md): Allows to very slightly actuate the switch, but not enough for sustained actuation
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md): Allows to break a frozen geizer
- An [Enemy](../../Entities/NPCControl/Enemy.md): Allows to move the enemy into a dizzy state
- Any NPCControl whose entity's name has the `ITAH` [modifier](../../Entities/EntityControl/Modifiers.md): Allows for the NPCControl to call [Interact](../../Entities/NPCControl/Notable%20methods/Interact.md)
- [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md): Allows to make the spring bounce
- Any ShakeHorn: Allows to shake the object

### Dash
This ability is very similar to horn slash detailed above, but the player is `dashing` which means the collider now stays in front of the player as long as the `dashing` is in effect.

This is the NON upgraded version of the dash, the upgraded version is detailed in the section below.

It means that while the colliders effects are the same, the difference comes from the player `dashing` which causes a complete override of most inputs processing and [Movement](Movement.md) by [DashBehavior](DashBehavior.md) which uses its own movement tracking fields such as `dashdelta`. Overall, movement inputs are more delayed and harder to control because the direction vector is based on a lerp of where the movement inputs are pointing towards where the factor is relatively small.

Here is what changes when the player is `dashing`:

- The [CloseMove](../MapControl/Follower%20system.md#closemove) logic never applies on followers
- [UpdateVelocity](../Entities/EntityControl/Update%20process/UpdateVelocity.md) ensures that the entity.`rigid` y velocity is clamped from -20.0 to `jumpheight` * 1.5
- The act of placing an [Enemy](../Entities/NPCControl/Enemy.md) in a dizzy state doesn't happen because while the collider allows it, it doesn't take effect when the player is `dashing`
- entity.`speed` is always `basespeed` * 2.5. Since `basespeed` should always be 5, it means the entity.`speed` is always 12.5 while it normally would have been 6.5 (when no dynamic friction is on the `ccol`) or 7.8 (when there is dynamic friction on the `ccol`)

### Horn dash
This ability is almost identical to the dash, but it is upgraded because the only difference besides some visuals changes is the collider tag is not `BeetleHorn`, but `BeetleDash`. The player `dashing` still functions the same way so the only change is the collider tag which generally provides more benefits.

This collider includes all of the effects of `BeetleHorn` and all of the effects of `dashing` with the following changes:

- [BreakableRock](../../Entities/NPCControl/ObjectTypes/BreakableRock.md): Allows to destroy the rock instead of just making it shake
- The act of placing an [Enemy](../Entities/NPCControl/Enemy.md) in a dizzy state can now happen even while `dashing` and if they were frozen, they are immeditately unfrozen
- Any NPCControl whose entity's name has the `ITHD` [modifier](../../Entities/EntityControl/Modifiers.md): allows for the NPCControl to call [Interact](../../Entities/NPCControl/Notable%20methods/Interact.md). It now means that both `ITAH` and `ITHD` modifiers are recognised

### Beetle dig
This ability allows to ignore certain colliders and it also include some [NPCControl](../Entities/NPCControl/NPCControl.md) changes as well as some movement changes. Actuating it will set `startdig` to true which will make FixedUpdate decrease the entity.`sprite` local y position so it reaches -2.0. Only when it reaches -1.5 will `digging` be set to true and include all the abilities's benefits and changes.

Here's what a `digging` does:

- It avoids detection by a DeadLanderOmega
- It avoids detection by an NPCControl with [StealthAI](../Entities/NPCControl/ActionBehaviors/StealthAI.md)
- [DigWall](../MapControl/DigWall.md) colliders gets disabled so it allows to pass through them
- [NPC](../Entities/NPCControl/NPC.md) and [Enemy](../Entities/NPCControl/Enemy.md) never have their `inrange` beahvior take effect, only their default one takes effect
- It avoids [Event](../Enums%20and%20IDs/Events.md) 116 (getting crushed) to trigger by colliding with a [RollingRock](../Entities/NPCControl/ObjectTypes/RollingRock.md)
- A `ToeBitter` [enemy](../Entities/NPCControl/Enemy.md) has its [ShootProjectile](../Entities/NPCControl/ActionBehaviors/ShootProjectile.md) change such that it yields all frames until the player stops `diggin` so they cannot throw a rock until that happens
- [CameraChange](../Entities/NPCControl/ObjectTypes/CameraChange.md) collisions are ignored and they will not cause any camera changes
- Ending the dig by releasing the ability input sets `uproot` to true for 0.1 seconds which reveals the hidden object under a [DigSpot](../Entities/NPCControl/ObjectTypes/DigSpot.md)
- The `spd` movement speed used by [Movement](Movement.md) is divided by 1.5 from regular movement logic

## `Moth` abilities
This section will detail `Moth`s field abilities. They have 3 abilities available.

### Freeze
This ability creates an `Icecle` tagged trigger collider that lives for only 1.0 seconds, but it has special properties because it is recognised by many other colliders in the game. Here is how these colliders reacts when colliding with a `Icecle` tagged collider:

- [SavePoint](../../Entities/NPCControl/ObjectTypes/SavePoint.md): Allows to interact with the save to save the game (doing so will call Kill on the `Icecle`'s DestroyOnLayer)
- [CoiledObject](../../Entities/NPCControl/ObjectTypes/CoiledObject.md): Allows to uncoil the object trapped
- [Switch](../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../Entities/NPCControl/ObjectTypes/WaterSwitch.md): Allows to actuate all of these switches
- [Dropplet](../../Entities/NPCControl/ObjectTypes/Dropplet.md): Allows to `hit` the dropplet which turns it into an ice cube
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md): Allows to freeze the geizer
- Any [Enemy](../../Entities/NPCControl/Enemy.md): Allows to freeze the enemy or to renew an existing freeze
- [JumpSpring](../../Entities/NPCControl/ObjectTypes/JumpSpring.md): Allows to make the spring bounce

### Icicle
This ability creates a `Prefabs/Objects/icecle` GameObject with the `icefall` tag with a RigidBody that has its gravity enabled with a y position of 4.0. It will fall down until colliding with layer 8 (`Ground`) or 13 (`NoDigGround`) trigger collider where the GameObject gets destroyed.

Due to the `icefall` tag and the presence of a BoxCollider on the GameObject, it has special properties because it is recognised by many other colliders in the game. Here is how these colliders reacts when colliding with a `icefall` tagged collider:

- `Water` type Hazards: Creates a `Prefabs/Objects/iceplatform` GameObject that acts as a frozen platform
- [Switch](../Entities/NPCControl/ObjectTypes/Switch.md), [StencilSwitch](../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and [WaterSwitch](../Entities/NPCControl/ObjectTypes/WaterSwitch.md): Allows to actuate all of these switches
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md): Allows to freeze the geizer

### Bubble shield
This abilities allows to reveal the `bubbleshield` GameObject which is a `Prefabs/Objects/BubbleShield` GameObject that lives on PlayerControl.

The GameObject is untagged, but the ability also sets the player to be in a `shield`. This changes some collisions logic:

- The player party and their followers will [Follow](../Entities/EntityControl/Notable%20methods/Follow.md) the player much more closely than usual
- GlowTrigger's `hbox` collider changes to be offscreen which prevents their `elecharz` Hazards component from colliding with the player
- `WalkableSpike` Hazards have their `col` center moved offscreen so the Hazards cannot be collided with
- Prevents to start a battle with an [Enemy](../Entities/NPCControl/Enemy.md) and instead, it places them in a dizzy state
- Prevents any OverworldProjectile launched due to any [ShootProjectile](../Entities/NPCControl/ActionBehaviors/ShootProjectile.md) to start a battle
- The `spd` movement speed used by [Movement](Movement.md) is divided by 1.5 from regular movement logic
