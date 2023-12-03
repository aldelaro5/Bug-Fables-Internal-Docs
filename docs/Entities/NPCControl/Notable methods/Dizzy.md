# Dizzy
Dizzy is a coroutine that performs the procedure done whenever an [Enemy](../Enemy.md) gets stunned in the overworld. It receives the amount of frames to stun it (set to `dizzytime`), a push force Vector3 (only the x and z components are considered) and whether or not we want to induce a vertical launch upward.

A yield break is performed immediately if the entity is `dead`, `iskill` or it has a `deathcoroutine` in progress.

First, [LockRigid(false)](../EntityControl/EntityControl%20Methods.md#lockrigid) is called on the entity which unlocks its `rigid`. 

The `disguiseobj` is disabled if it exists with the entity.`sprite` getting enabled.

From there, there's a special case if the entity's `originalid` is the `ToeBiter` [animid](../../Enums%20and%20IDs/AnimIDs.md) where if the `internaltransform[0]` is present (the boulder he throws), it needs to be setup for its destruction. This is first done by adding a RigidBody to it, make it go in a random direction of 3.0 magnitude at a height of 12.0 and destroying it in 3.0 seconds. The entire `internaltransform` array is then set to null.

From there, a couple of adjustements happens:
- The `touchcooldown` is set to 30.0 frames
- The `dizzytime` is to the sent time parameter
- The entity.`sprite` has its angles reset to Vector3.zero
- The entity.`onground` is set to false
- STOP is called which does the following:
  - Stop `behaviorroutine` if it was in progress and set it to null
  - Set `forcebehavior` to null and call [StopForceBehavior](StopForceBehavior.md)
  - Call [StopForceMove](../EntityControl/EntityControl%20Methods.md#StopForceMove) on the entity
  - Set the entity.`overrideflip` and entity.`overrideonlyflip` to false
  - Set `attacking` to false

After, a frame is yielded.

If the enemy has an [Unmoveable](../ActionBehaviors/Unmoveable.md) behavior, it cuts off most of the rest of the coroutine only limiting itself to play the `hitpart` particle at the `sprite` position + (0.0, 0.5, 0.0) and yielding for a frame.

## Not Unmovable
If it doesn't have this behavior however, the rest of the logic is performed:
- Set `ignoreconstraint` to true
- Enable gravity on the entity.`rigid` without kinematic
- If the entity didn't had a `detect`, [CreateDetector](../EntityControl/EntityControl%20Methods.md#CreateDetector) is called to create it
- The entity.`detect` is set to look at the player and its y angle is multiplied by -1
- The entity.`hitwal` is set to false
- All rotation on the entity.`rigid` are frozen
- The actual pushforce vector is comnputed from the the x and z components of the one sent in, but the y one is 0.0 unless we asked for a vertical launch which means the y is 13.5 instead
- The entity.`rigid` velocity is set to the push force computed above
- If the magnitude of that push force vector is above 0.1, TempIgnoreColision is called on the entity ignoring all collisions between the entity.`ccol` and entity.`detect` with the player's corresponding colliders for 0.5 seconds

After, the `hitpart` particle is played at the entity.`sprite` position + (0.0, 0.5, 0.0) and a frame is yield.

Finally, if the entity.`rigid` y velocity is less than 5.0, [Jump](../EntityControl/EntityControl%20Methods.md#jump) is called on the entity with a velocity of 13.5 for every frame as until the entity.`rigid` y velocity reaches 5.0 or above.