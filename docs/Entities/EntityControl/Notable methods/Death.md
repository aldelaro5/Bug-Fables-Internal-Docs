# Death
Death is a coroutine in EntityControl that handles the process of essentially rendering an entity effectively dead. For most cases, it leads to its destruction. It comes with a parameterless overload that has `activatekill` sent to true, that parameter tells if `iskill` should be set to true alongside `dead` (however, the game never calls it with false under normal gameplay). The coroutine is meant to be stored by the caller using `deathcoroutine` which is set to null when this coroutine ends.

Essentially, `dead` means the Death process started while `iskill` means it is about to end (it's done just before the `spitexp` logic). It will also in most cases leads to the destruction of the entity, but these 2 fields can be set externally without calling Death which will reduce their updating logic throughout the game, but not cause destructions.

## Destroy prepations
First, `nocondition` and `dead` are set to true. [BreakIce](Freeze%20handling.md#breakice) is called and [StopForceMove](../EntityControl%20Methods.md#stopforcemove) is too without smoothing and default state. This not only unfreezes the entity if it was, but also stops any coroutine force move that was running. After, the `rigid`'s velocity is zeroed out and the `digpart` destroyed if there were any left.

From there, if the entity had an `npcdata`, it is handled in its own section:
- STOP is called on the npcdata, but this doesn't do anything because `dead` was just set to true
- If npcdata.`regionalflag` isn't negative, the corresponding [regionaflag](../../../Flags%20arrays/Regionalflags.md) slot is set to true
- If npcdata.`disguiseobj` exists, it is destroyed
- If npcdata.`behaviorroutine` is running, it is stopped
- npcdata.`inrange` is set to false
- npcdata.`hit` is set to true
- If `originalid` is the `ToeBiter` [animid](../../../Enums%20and%20IDs/AnimIDs.md) and the first `internaltransform` (his rock) exists, it is destroyed

After, `overrideflip` is set to false, the `sprite` gets enabled, the `rigid` gets locked with a LockRigid(true) and the `icooldown`, `bobrange` and `bobspeed` all get set to 0.0.

## The destroy
This part depends on the `destroytype`.

### None
LockRigid(true) is called which fixes the `rigid` in place with velocity being zeroed out.

### SpinSmoke
- If the npcdata.`pusher` exists, it is disabled.
- `overrideanim` is set to true
- `overridefollow` is set to true
- `overridejump` is set to true
- `rigid` gets its gravity disabled and velocity zeroed out
- `ccol` gets its height set to 0.0 and center placed offscreen at (0.0, 9999.0. 0.0)
- A frame is yielded
- If the [animstate](../Animations/animstate.md) is `Fallen`, it is set to `HurtFallen`. It is set to `Hurt` otherwise.
- `froceanim` is set to the `animstate`
- If the `Death0` sound wasn't playing or it was, but the playback is past 0.25 seconds, it is played at 0.8 volume
- `spin` is set to (0.0, 15.0, 0.0)
- 0.75 seconds are yielded
- The `spritetransform` angles are set to the return of FlipAngle
- `spin` is set to (-5.0, 0.0, 0.0)
- 0.3 seconds are yielded
- The `sprite` is disabled
- DeathSmoke particles are played at the `spritetransform` position + (0.0, 0.0, -0.5)

### SpinNoSmoke
The same than SpinSmoke, but without the DeathSmoke particles at the end.

### SpinKO
The same than SpinSmoke, but the horzontal `spin` logic (with its yield, `sprite` disablement and DeathSmoke particles) does not occur, only the vertical `spin` does. Also, the `animstate` at the very end is set to `KO`.

### KO
The same than SpinKO, but the vertical `spin` logic (with its yield and `Death0` sound) does not occur too.

### SpinSmokeNoSprite
The same than KO, but at the start, [animid](../../../Enums%20and%20IDs/AnimIDs.md) is set to -1 (`None`) and the `sprite` is disabled. The `animstate` is also not changed to `KO` at the end.

### PlayerDeath
This type is only used in BattleControl.StartBattle as it is set to all `playerentity` so it's specific to the death of the player.

- `overrideanim` is set to true
- [animstate](../Animations/animstate.md) is set to `Hurt`
- The `Hurt` animation is played on the `anim`
- If the `Death0` sound wasn't playing or it was, but the playback is past 0.2 seconds, it is played
- `spin` is set to (0.0, 15.0, 0.0)
- While `spin` magnitude is above 2.5, it is lerped from the existing one to 0 with a factor of `framestep` * 0.1 (or 0.01 if `isplayer`). This is followed by a frame yield which repeats until the `spin` magnitude goes to 2.5 or below
- `overridenanim` is set to false
- The `Drop` sound is played
- `animstate` is set to `KO`
- `spin` is set to (0.0, 0.0001, 0.0)
- While the angular distance between `spritetransform` angles and the return of FlipAngle is above 0.5, `spriteransform` angles is lerped from the existing one to the FlipAngle one with a factor of `framestep` * 0.2 (or 0.1 if `isplayer`). This is followed by a frame yield which repeats until the angular distance goes to 0.5 or below
- `spin` is zeroed out

### DropSprites
- `overrideanim` is set to true
- The `anim` is disabled
- A frame is yielded
- For each Transform descendant to the `sprite` (or the `model` if it exists) except the first one:
  - A RigidBody is added to the transform
  - The transform gets rooted to the scene
  - The RigidBody gets its gravity disabled with a velocity of RandomItemBounce(2.5, 12.0)
  - The object gets destroyed in a second
- A second is yielded

### Shrink
- LockRigid(true) is called which fixes the `rigid` in place with velocity being zeroed out
- If `originalid` is the `icepillar` [animid](../../../Enums%20and%20IDs/AnimIDs.md), the `IceMelt` sound is played
- If `originalid` is the `Pitcher` or `PitcherSummon` [animid](../../../Enums%20and%20IDs/AnimIDs.md), the [animstate](../Animations/animstate.md) is set to `Hurt` followed by the `IceMelt` sound being played
- For 81.0 frames (counted by incrementing `framestep` from 0.0), the `startscale` is set to a lerp from the existing one to Vector3.zero with a factor of the ratio of the 81.0 frames that have passed (so the `startscale` goes to zero over the course of 81.0 frames smoothly)
- DeathSmoke particles are played at the entity position
- If `originalid` is the `Pitcher` or `PitcherSummon` [animid](../../../Enums%20and%20IDs/AnimIDs.md), the `ChargeDown2` is played

### ShrinkNoSmoke
The same than Shrink, but no DeathSmoke particles are played.

### NinjaLog
- LockRigid(true) is called which fixes the `rigid` in place with velocity being zeroed out
- DeathSmoke particles are played at the entity position with a size of (2.0, 3.0, 2.0)
- The `LeafDeathPoof` sound is played

### Sink
- `overridefly` is set to true
- `overrideflip` is set to true
- `overrideheight` is set to true
- As long as the `sprite` exists, `spritetransform` position is lerped smoothly over the course of 91.0 frames (counted using `framestep` from 0.0) from the existing one to the same - 10.0 in y using the ratio of the time as the factor (this basically lowers the entity by 10.0 smoothly over the ourse of 91.0 frames)
- If the `sprite` still exists, the gameObject is disabled

### ExplodeAnim
- `explosionsmall` particles are played at the entity position
- `Explosion` sound is played at 1.1 pitch and 0.75 volume
- The `sprite` is disabled
- ShakeScreen is called with ammount of 0.1 for 0.5 seconds without reset

## After the `destroytype` specific logic (doesn't occur for `PlayeDeath`)
- All frames are yielded while in a `pause` or it's not a `battle` entity while we are in a battle
- For anything except an `npcdata` of type [Enemy](../../NPCControl/Enemy.md) with an `eventid` of 0 or below (meaning no `respawntimer` feature):
  - If `spitmoney` is above 0, the berries drop logic is performed (see the section below for details)
  - If `npcdata` has a non empty `vectordata` without a [SetPath](../../NPCControl/ActionBehaviors/SetPath.md) or [SetPathJump](../../NPCControl/ActionBehaviors/SetPathJump.md) behaviors, the item drop logic is performed (see the section below for details)
- A frame is yielded
- If this GameObject is null (which shouldn't happen), the coroutine is exited early with a yield break
- If the `destroytype` isn't `KO`, `SpinKO` or `None`, the entity position is set offscreen at (0.0, 9999.0, 0.0) followed by the destruction of the object (only if we aren't in a battle)
- The `ccol` is disabled

### Berries drop logic
The following is performed until `spitmoney` amount of berries worth total have been dropped:
- [CreateItem](../../NPCControl/ObjectTypes/Item.md#entitycontrolcreateitem) is called which creates an [Item](../../NPCControl/ObjectTypes/Item.md) NPCControl object at `spritetansform` position + (0.0, 0.5, 0.0) with the item type being 0 (standard item), the direction being RandomItemBounce(4.0, 10.0) for 600 frames. The [item](../../../Enums%20and%20IDs/Items.md) id is `MoneyBig` if there's strictly more than 20 berries left to drop, `MoneyMedum` if there's 5 or less left and `MoneySmall` otherwise (this means the last 20 berries if exactly 20 are left will be dropped by 4 `MoneyMedum` instead of one `MoneyBig`)
- The collisions between the item's entity.`ccol` and the item's entity.`detect` or itself are ignored for 5.0 seconds
- The same RandomBounce vector obtined earlier is set to the item's entity.`rigid` vecity on the next frame
- The amount of money dropped is increased by 20 if there was more than 20 left to drop. Otherwise, if there's more than 5 left, it's increased by 5 (this means the last 5 berries if exactly 5 are left drops 5 `MoneySmall` instead of one `MoneyMedum`)

### Item drop logic
The `specialenemy` cases are handled. These are hardcoded [enemy](../../../Enums%20and%20IDs/Enemies.md) ids with hardcoded odds to drop an [item](../../../Enums%20and%20IDs/Items.md) from an hardcoded list of ids with uniform probability each. The way it works is the first occurence of a special enemy in `lastdefeated` (if one exists) will test for a potential drop. If multiple exists, only the first is tested once so if it fails, others will not be attempted to drop. Here are the the hardcoded ids in question as well as their odds:
|Enemy|% to drop|Possible item drops|
|-----|---------|-------------------|
|`GoldenSeedling`|100|`TangyBerry`|
|`ChomperBrute`|50|`Abomihoney`, `ShockCandy`, `DrowsyCake`, `FrostPie`, `PoisonCake`, `MushroomCandy`, `NutCake`|
|`ToeBiter`|40|`HoneydLeaf`, `GlazedHoney`, `HeartyBreakfast`, `YamBread`, `BakedYam`, `Pudding`, `RoastBerry`, `ClearBomb`, `SleepBomb`, `LeafSalad`, `FrozenSalad`, `MushroomStick`, `ShavedIce`, `BurlyChips`|

If a drop occurs:
- [CreateItem](../../NPCControl/ObjectTypes/Item.md#entitycontrolcreateitem) is called which creates an [Item](../../NPCControl/ObjectTypes/Item.md) NPCControl object at `spritetansform` position + (0.0, 0.5, 0.0) with the item type being 0 (standard item), the item id being a randomly chosen (uniform odds) element from the applicable possible drop list, the direction being RandomItemBounce(4.0, 10.0) for 600 frames.
- The collisions between the item's entity.`ccol` and the item's entity.`detect` or itself are ignored for 5.0 seconds
- The same RandomBounce vector obtined earlier is set to the item's entity.`rigid` vecity on the next frame

Whether or not the drop failed or succeeded, the enemy is removed from the `lastdefeated` array. From there, `lastdefeated` is reset to a new list (making the last deletion useless).

An random number is generated where the upper (exclusive) bound is the length of npcdata.`vectordata`, but the lower (inclusive) bound is determined based on some factors:
- Having the Bug Me Not! [medal](../../../Enums%20and%20IDs/Medal.md) equipped makes it -7 (this takes priority over the ones below)
- Having the Hard Mode medal unequipped while [flag](../../../Flags%20arrays/flags.md) 614 (HARDEST) is false makes it -3
- If neither of the cases above applies (meaning Hard Mode is equipped or HARDEST is active while Bug Me Not! is unequipped), it's -1

However, that index gets overriden if `vectordata` contains at least one element whose y component after truncating the decimal part is -2. In such case, it will be become the index of the last occurence where this condition is true.

This index is used for a potential item drop. If the index is negative, no drops happen. If it's positive, but the y component of the corresponding npcdata.`vectordata` isn't negative, then the drop only happens if that y component floored corresponds to a [flag](../../../Flags%20arrays/flags.md) slot whose value is true. If it's false, the drop doesn't happen.

If the drop happens:
- [CreateItem](../../NPCControl/ObjectTypes/Item.md#entitycontrolcreateitem) is called which creates an [Item](../../NPCControl/ObjectTypes/Item.md) NPCControl object at `spritetansform` position + (0.0, 0.5, 0.0) with the item type being 0 (standard item), the item id being the chosen npcdata.`vectordata` x component using the index generated earlier, the direction being RandomItemBounce(4.0, 10.0) for 600 frames.
- The collisions between the item's entity.`ccol` and the item's entity.`detect` or itself are ignored for 5.0 seconds
- The same RandomBounce vector obtined earlier is set to the item's entity.`rigid` vecity on the next frame

However, if the corresponding npcdata.`vectordata` y component is -2 (meaning its index was overriden earlier), then it means this is a special key item drop that will always be dropped. The procedure to drop it is the same, but with a few changes:
- The item type is 1 (key item) instead of 0 (standard item)
- -1 is the timer value which means the [Item](../../NPCControl/ObjectTypes/Item.md) never expires even after 600 frames
- After the [CreateItem](../../NPCControl/ObjectTypes/Item.md#entitycontrolcreateitem) call, the item's `activationflag` is set to npcdata.`limit[0]`

## Common end logic
The following happens no matter the `destroytype`.

A frame is yielded.

After, there is a specific case for the Ruffian [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) where if it is one, it will destroy all `extras` and destroy the MidPos of the `sprite` that was attached prior before setting `extras` to null.

After, this is where `iskill` is set to true if the `activatekill` parameter was true. This field is more an extended `dead` and while it only affects [UpdateCollider](../Update%20process/UpdateCollider.md) in [EntityControl Creation](../EntityControl%20Creation.md), it can be used externally to distinguish the 2.

Then, if `spitexp` is above 0, this is where the orbs gets dropped. The way it works is the game calculates the amount of big orbs this implies where a big orb is 10 EXP. Then, `spitexp` is essentially decreased by this amount * 10 which is equivalent to do a modulo 10 on it. From there, the game will set to drop `spitexp` amount of orbs where any big orbs will take priority if any are remaining to be dropped. (NOTE: this is a bug, it means not all orbs will be dropped, see below for why). An orb drop consists of instantiating `Prefabs/Objects/ExpOrb` from the root of the ressources tree, set its scale to (0.25, 0.25, 0.25) or (0.5, 0.5, 0.5) if it's a big orb, adding a rigid body to it with velocity being 15.0 in y and the x/z being random between -4.0 and 4.0. The orb is then destroyed in 0.75 seconds, but the coroutine yields only 0.05 seconds after so it gives some time to see the orb drop.

The reason the orb drop is wrong is because it will only drop `spitexp` % 10 amount of orbs no matter how many big orbs are meant to be dropped. This means that a value of 10 will actually drop nothing while 11 will drop one big orb.

Finally, `deathcoroutine` is set to null informing the caller that it has completed.
