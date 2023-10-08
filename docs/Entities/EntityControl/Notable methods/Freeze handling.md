# FreezeHandling

Entities can get frozen which involves the entity being put in the `icecube` and unfrozen which involves breaking the cube and restoring the entity's functions. This is done by 2 methods called Freeze and BreakIce respectively.

## Freeze

First, `freezeoffset` is set to `initialoffset`, but the x component is multiplied by -1 if `flip` is true. Unless it is a `battle` entity, the `Freeze` sound will play unless it was already playing.

From there, this is where the `icecube` is prepared to be frozen. The `icecube` is instantiated from `icecubeprefab` childed to the root `transform` with a local scale of zero. The scale is left handled by [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md) which will grow it to `freezesize`.

For the entity, its `onground` is forced to true, [StopForceMove](../EntityControl%20Methods.md#StopForceMove) is called with `spin`, `extraoffset` and the `anim`'s speed all zeroed out with the [animstate](../Animations/animstate.md) set to `Hurt`. Since the ice cube has a collider, collisions with the `ccol` are ignored to avoid problems. If it's not a `battle` entity and there is an `npcdata`, STOP is called on it. Finally, the SpriteBounce is disabled from the `rotater` if one was present.

Before returning, [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) is called since some [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) have special handling when the entity is frozen.

## BreakIce

This undo everything that happened on Freeze with some more logic related to it.

First, the destruction of `icecube` if it was still present. This occurs by first playing the `Prefabs/Particles/IceShatter` particles from the root of the resources tree at the `icecube`'s position along with the `IceBreak` sound with 0.65 of pitch. The cube is now destroyed with the entity's [animstate](../Animations/animstate.md) set to `basestate`, the `anim`'s speed to 1.0 and `onground` being set back to false with a Jump.

Them, the `height` logic follows. `tempheightoverride` is set to false which is needed in the case a [Drop](Drop.md) occured. From there, unless `overrideminheight` is true, the `minheight` is enforced if the `height` is less than it which also comes with the `bobrange` and `bobspeed` being set to their start counterparts.

If it's a `battle` entity that isn't the `player`, its size in the battle data is set to the initialsize of the battle data.

Finally, `shakeice` is set to false and the SpriteBounce is enabled back if one existed on the `rotater`. Before return, [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) is invoked in 0.1 seconds.
