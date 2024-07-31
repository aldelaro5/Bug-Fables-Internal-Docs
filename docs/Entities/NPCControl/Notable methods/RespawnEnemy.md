# RespawnEnemy
This is a helper method to respawn an [Enemy](../Enemy.md) at a certain position. It takes in the NPCControl and the Vector3 position. The position will get incremented by Vector3.up * 0.5.

- entity.`destroytype` is set to `SpinSmoke`
- `attacking` is set to false
- `behaviorroutine` is stopped and set to null
- All coroutines on the entity are stopped with entity.`deathcoroutine` set to null
- entity.`dead` and entity.`iskill` are set to false
- entity.`spin` is zeroed out
- entity.`rigid` gravity is enabled without kinematic and with the velocity zeroed out
- entity.`sprite` is enabled and its angles set to zero
- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to the `basestate`
- entity.`overrideanim` is set to false
- entity.`onground` is set to false
- entity.`bobspeed` and entity.`bobrange` are reset to their start fields counterparts
- entity.`lastpos` is set to the position passed as parameter
- entity.`pausepos` is set to null unless we are `paused` in which case, it is also set to the passed position
- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called
- `dizzytime` is set to 0.0
- `respawntimer` is set to -100.0
- `freezecooldown` is set to 0.0
- Some [SetPath](../ActionBehaviors/SetPath.md) exclusive logic occurs here
- entity.`ccol` is enabled in without trigger
- `hit` is set to false
- The position and the entity's `startpos` are set to the passed parameter
- Some logic happens if the default behavior is a disguise one
- [LockRigid(false)](../../EntityControl/EntityControl%20Methods.md#lockrigid) is called on the entity which unlocks its `rigid`
- The `pusher` is enabled if it exist. NOTE: normally, enemies do not have one
- DeathSmoke is called on the entity.`sprite`'s position