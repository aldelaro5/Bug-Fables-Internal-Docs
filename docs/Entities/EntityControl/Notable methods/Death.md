# Death

Death is a coroutine in EntityControl that handles the process of essentially rendering an entity effectively dead. It doesn't disable or destroy the entity, it only does the necessary logic to effectively render it non functional. It comes with a parameterless overload that has `activatekill` sent to true, that parameter tells if `iskill` should be set to true alongside `dead`. The coroutine is meant to be stored by the caller using `deathcoroutine`.

## Destroy prepations

First, `nocondition` and `dead` are set to true. [BreakIce](Freeze%20handling.md#breakice) is called and [StopForceMove](../EntityControl%20Methods.md#stopforcemove) is too without smoothing and default state. This not only unfreezes the entity if it was, but also stops any coroutine force move that was running. After, the `rigid`'s velocity is zeroed out and the `digpart` destroyed if there were any left.

From there, if the entity had an `npcdata`, it is handled in its own section. TODO: Come back to this when [NPCControl](../../NPCControl/NPCControl.md) is documented.

After, `overrideflip` is set to false, the `sprite` gets enabled, the `rigid` gets locked with a LockRigid(true) and the `icooldown`, `bobrange` and `bobspeed` all get set to 0.0.

## The destroy

This part depends on the `destroytype`. TODO: Come back to this section after [NPCControl](../../NPCControl/NPCControl.md) is documented.

## After the destroy

There is a specific case for the Ruffian [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) where if it is one, it will destroy all `extras` and destroy the MidPos of the `sprite` that was attached prior before setting `extras` to null.

After, this is where `iskill` is set to true if the `activatekill` parameter was true. This field is more an extended `dead` and while it only affects [UpdateCollider](../Update%20process/UpdateCollider.md) in [EntityControl Creation](../EntityControl%20Creation.md), it can be used externally to distinguish the 2.

Then, if `spitexp` is above 0, this is where the orbs gets dropped. The way it works is the game calculates the amount of big orbs this implies where a big orb is 10 EXP. Then, `spitexp` is essentially decreased by this amount * 10 which is equivalent to do a modulo 10 on it. From there, the game will set to drop `spitexp` amount of orbs where any big orbs will take priority if any are remaining to be dropped. (NOTE: this is a bug, it means not all orbs will be dropped, see below for why). An orb drop consists of instantiating `Prefabs/Objects/ExpOrb` from the root of the ressources tree, set its scale to (0.25, 0.25, 0.25) or (0.5, 0.5, 0.5) if it's a big orb, adding a rigid body to it with velocity being 15.0 in y and the x/z being random between -4.0 and 4.0. The orb is then destroyed in 0.75 seconds, but the coroutine yields only 0.05 seconds after so it gives some time to see the orb drop.

The reason the orb drop is wrong is because it will only drop `spitexp` % 10 amount of orbs no matter how many big orbs are meant to be dropped. This means that a value of 10 will actually drop nothing while 11 will drop one big orb.

Finally, `deathcoroutine` is set to null informing the caller that it has completed.
