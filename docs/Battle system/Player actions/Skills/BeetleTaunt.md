# `BeetleTaunt`

## Explanation of how taunt works
This action has a special interaction with [GetRandomAvailablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) where it will force the target selection by setting `forceattack` to `currentturn` (meaning the attacker performing this action). 

While the enemy party members gets the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition (if they didn't had [notaunt](../../Actors%20states/Enemy%20features.md#notaunt) enabled), the condition doesn't actually do anything logically and it only visuals. 

What will influence the battle logically solely has to do with `forceattack` being set. This action only features that and the animations related to it.

## Logic sequence

- attacker animstate set to 109
- `dontusecharge` set to true
- `Taunt2` sound played on `sounds[9]` at 0.9 pitch on loop
- Camera positioned to the attacker a bit up and back
- Yield for 1.25 seconds
- Camera reset to default
- For every enemy party member whose [notaunt](../../Actors%20states/Enemy%20features.md#notaunt) is false:
    - `Prefabs/Particles/AngryParticle` instantiated and childed to the enemy party member positioned at their world [CenterPos](../../Actors%20states/CenterPos.md) + -0.1 on z
    - enemy party member's `overridejump` set to true with the old value saved temporarilly
    - SetCondition called to add the [Taunted](../../Actors%20states/BattleCondition/Taunted.md) condition on the enemy party member for 1 turn
    - If `TauntPlus` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped: 
        - SetCondition called to add the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the enemy party member for 1 turn
        - SetCondition called to add the [DefenseDown](../../Actors%20states/BattleCondition/DefenseDown.md) condition on the enemy party member for 1 turn
- `sounds[9]` is stopped
- For the next 100.0 frames and for each enemy party member whose [notaunt](../../Actors%20states/Enemy%20features.md#notaunt) is false, the `Taunt` sound may be played if it wasn't playing already (or it is, but its time is past 0.25 seconds) under 2 conditions:
    - Every 20.0 frames while the enemy party member's `weight` is 100.0 or higher
    - The enemy party member's `weight` is less than 100.0, it is `onground`, its `jumpcooldown` expired and a 10% RNG test passes (this will also have [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on the entity party member before the sound playing)
- For each enemy party member:
    - Their `Prefabs/Particles/AngryParticle` is destroyed
    - Yield until the enemy party member is `onground`
    - The enemy party member's `overridejump` is reverted to its initial value it had at the start of the action
- `forceattack` is set to `currentturn`
