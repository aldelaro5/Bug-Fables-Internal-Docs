# Submarine
There is a field that relates to a feature in PlayerControl where the player can be in a `submarine`, but it is implemented in a rather complex way.

## Enterting and exiting the `submarine`
While the field is public and can be toggled by the rest of the game, it's not enough to actually setup this feature. The only place in the entire game that set it up is [Event](../Enums%20and%20IDs/Events.md) 153 (interacting with the submarine or a location while in the submarine) when choosing to board it. PlayerControl has strong assumptions that the procedure this event does already happened and it consists of the following:

- Sets player.`submarine` to true
- Make a new `Prefabs/Objects/Submarine` GameObject childed to the player.entity.`sprite` with a uniform scale of 0.1 and angles of (-90.0, 180.0, 0.0)
- Disables the player.entity.`sprite`
- Sets the player.entity.`model` to the `Prefabs/Objects/Submarine` GameObject created above
- Sets player.entity.`extras` to be 2 elements where the first element is the first child of the `Prefabs/Objects/Submarine` GameObject and the second element is its second child

These steps are all that should be required to enter the `submarine`. To exit it, the same event does the following when unboarding it:

- Sets player.`submarine` to false
- Destroys the first child of player.entity.`sprite` (which is the `Prefabs/Objects/Submarine` GameObject)
- Enable player.entity.`sprite`

## Effects of being in a `submarine`
There are several effects that happens when being in a `submarine`. The most notable one is it changes the semantic of the `digging` field because it now indicates if the player is underwater.

The underwater ability can be toggled via [DoActionTap](Actions/DoActionTap.md) which also completely changes when in a `submarine` to simply toggle `digging`. Because the field is reused for this, all the effects of [Beetle dig](Field%20abilities.md#beetle-dig) applies which in theory can lead to unexpected behaviors, but in practice, there's only one effect that matters: [NPC](../Entities/NPCControl/NPC.md) and [Enemy](../Entities/NPCControl/Enemy.md) never have their `inrange` beahvior take effect, only their default one takes effect.

Being in a `submarine` also have other changes:

- The player entity, through [EntityControl's Update](../Entities/EntityControl/Update%20process/Unity%20events/Update.md) gets its `sprite` disabled when active
- The [chompy follower](../MapControl/Follower%20system.md#chompy-following) is never added on the map's first LateUpdate (known as `latestart`)
- The switch input isn't processed by [GetInput](GetInput.md)
- It is not possible to jump by using the jump / interact input, but it is still possible to [Interact](../Entities/NPCControl/Notable%20methods/Interact.md) with an NPCControl with that input
- It is not possible to do any [DoActionHold](Actions/DoActionHold.md)
- On FixedUpdate, the `model`, `extra` and `sprite` of the player entity gets adjusted to align with the movement of the `submarine` and whether is it underwater or not
- On LateUpdate and when not underwater, the `Submarine` sound is set to play on loop on the entity with a pitch that depends on the entity.`rigid` velocity's magnitude (changes with movement)
- On [Movement](Movement.md) and when underwater, the `Sonar` sound plays on the entity at 0.75 volume if it wasn't playing already so it effectively keeps playing until going back to the surface, but it's not played on loop
- The `spd` movement speed used by [Movement](Movement.md) is divided by 1.8 from regular movement logic
