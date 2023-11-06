# RespawnEnemy
This is a helper method to respawn an `Enemy` at a certain position. It takes in the NPCControl and the Vector3 position. The position will get incremented by Vector3.up * 0.5.

In there, several entity adjustements occurs:
- The `destroytype` is set to `SpinSmoke`
- All coroutines on the entity are stopped with `deathcoroutine` set to null
- `isdead` and `iskill` are set to false
- The `spin` is zeroed out
- The `rigid` gravity is enabled without kinematic and velocity
- The `sprite` is enabled and its angles set to zero
- The [animstate](../EntityControl/Animations/animstate.md) is set to the `basestate`
- `overrideanim` is set to false
- `onground` is set to false
- The `bobspeed` and `bobrange` are reset to their start fields counterparts
- `lastpos` is set to the position passed as parameter
- The `pausepos` is set to null unless we are `paused` in which case, it is also set to the passed position
- [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) is called

Adjustements on the NPCControl happens too:
- `attacking` is set to false
- The `behaviorroutine` is stopped if one was ongoing and set to null
- `dizzytime` is set to 0.0
- `respawntimer` is set to -100.0
- `freezecooldown` is set to 0.0
<!-- - If the enemy has the `SetPath` behavior, the `actioncooldown` is set to 0.0 and the `currentnode` to 0 -->

From there, the following happens:
- The entity's `ccol` is enabled in without trigger
- `hit` is set to false
- The position and the entity's `startpos` are set to the passed parameter
<!-- - If the default behavior is a diguside one:
  - The `disguiseobj` is destroyed if it was present
  - The entity's `speed` is set to 2.0
  - The `disguisecooldown` is set to -1
  - The default behavior is set to `Wander`
  - The entity's `onground` is set to true in 0.5 seconds -->
- [LockRigid(false)](../EntityControl/EntityControl%20Methods.md#LockRigid) is called on the entity which unlocks its `rigid`
- The `pusher` is enabled if it exist. NOTE: normally, enemies do not have one
- DeathSmoke is called on the `sprite`'s position