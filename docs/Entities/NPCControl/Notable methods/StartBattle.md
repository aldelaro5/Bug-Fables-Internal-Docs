# NPCControl.StartBattle
This is a void and parameterless method in NPCControl (not to be confused with the coroutine in BattleControl) that aims to handle when a player engages with an [Enemy](../Enemy.md) which happens for example when getting too close to it. 

The name is a bit missleading here: while it does take part in the start of an actual battle, it is not the method that starts it. That would be the coroutine of the same name in BattleControl which this does start one if all conditions are met. This method doesn't just setup the start of the battle, it also performas any logic needed the moment an enemy is engaged which may include not doing anything or even do something else instead of starting a battle.

## Bubble Shield logic
If the player's `shield` is on and the entity's `originalid` isn't the `DeadLanderC` [animid](../../../Enums%20and%20IDs/AnimIDs.md) (it cannot be stunned using Bubble Shield) then this logic applies. It will only do anything if the `touchcooldown` expired, otherwise, nothing happens.

If the `ShieldHit` audio clip isn't playing on any of the `sounds` audio sources, it is played on the first free one with 75% volume and a random pitch between 0.95 and 1.05. This is followed by a [Dizzy](Dizzy.md) call for 100.0 frames with a pushforce being the direction vector from this enemy to the player * 5.0 without vertical launch.

## Main logic
This section happens only if the Bubble Shield one didn't.

2 special early exits cases are handled:

- If we are already in a `battle` or we are in a `minipause`, this causes this method to return early
- If `entitytouchevent` is 0 or above, the [event](../../../Enums%20and%20IDs/Events.md) whose id is this field is started before returning early. This is essentially a global feature of the game that allows to override most engagement with an enemy to be an event start instead of a potential battle.

If we haven't returned from this, the entity.`lastpost` is set to this enemy position and the entity.`rigid` velocity is zeroed out.

From there, 2 other early returns special cases are checked:

- If the `icooldown` of the player.entity hasn't expired yet or the enemy's `freezecooldown` hasn't expired yet or its entity is `dead` or its `touchcooldown` hasn't expired yet, this method will return early.
- If the the Bug Me Not! [medal](../../../Enums%20and%20IDs/Medal.md) applies, the entity is put in kinematic mode without velocity and a [Death(true)](../../EntityControl/Notable%20methods/Death.md) coroutine is started on it. The logic to determine if the medal applies is as follows:
    - The played must have it equipped or it will not apply
    - If the [flag](../../../Flags%20arrays/flags.md) slot 555 (post game) is true, the `partylevel` is exactly 27 and the current [map](../../../Enums%20and%20IDs/Maps.md) isn't `GiantLairDeadLands1` or `GiantLairDeadLands2`, then it applies no matter what (NOTE: it implies the medal never works on these 2 maps because the case below excludes the area that includes them too)
    - Otherwise, it will only apply if the [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) isn't Giant's Lair, the map isn't `BugariaPlazaAttack`, `BugariaBridgeAttack` or `BugariaCastleAttack` and either the `partylevel` is exactly 27 or the ExpEstimate of this encounter is equal or less than the amount of enemies in this encounter

If we still haven't returned, then a battle will occur and the rest is just to setup the StartBattle call.

All [EntityControl](../../EntityControl/EntityControl.md) will have their `lastpos` set to their current positions. Then, CancelAction is called on the player and the enemy's entity.`spin` is zeroed out. `attacking` is set to false and finally, [StopForceBehavior](StopForceBehavior.md) is called.

The last step is the actual StartBattle call:

- The enemies list is `battleids`
- Automatically determine the battle map
- No specific advantage except if `attacking` was true when this was called and the `dizzytime` expired while the player doesn't have the Crazy Prepared [medal](../../../Enums%20and%20IDs/Medal.md) equipped. In that case, it will be set to 3 with the `Damage0` sound playing
- The music is automatically determined unless the enemies list contains a `WaspTrooper`, `WaspBomber`, `WaspDriller` or `WaspHealer` which causes the music to be overriden to `Battle3`
- Caller is this enemy
- With the ability to run

