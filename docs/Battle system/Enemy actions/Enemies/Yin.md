# `Yin`

## Assumptions
This enemy is assumed to not be fought alone because if they are fought alone, they will always immediately kills themselves on their first actor turn which is unexpected.

## A note about misshandling of the `target` field
This enemy always writes to the `target` field, but they aren't supposed to because this field is reserved for the player party's targetting. The field is never supposed to be used for an enemy party member, but this one does and it can overwrite its value.

However, it turns out that this is inconsequential. Due to how [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) and [GotoSelect](../../Player%20UI/GotoSelect.md) works in the player UI system, it's not possible to poison DoAction by leaving `target` with an illogical value. There is an extremely narrow cause where the [HardCharge](../../Player%20actions/Skills/HardCharge.md) and [Sturdy](../../Player%20actions/Skills/Sturdy.md) actions may leave `target` to the value this enemy set, but doing so will have no effect on the logic of DoAction.

Effectively, while this field is used when it shouldn't on this eneny, it has no consequential impact on the rest of the battle system. The field that should have been used for this purpose is either a local or `playertargetID` which is designed to hold the enemy's targetting and it is constantly written to before usage for most enemies.

## [StartBattle](../../StartBattle.md) special logic
After loading this enemy, the following special adjustements happens to them if [flags](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active):

- `maxhp` and `hp` gets increased by 10
- `def` gets incremented

## Move selection
6 moves are possible:

1. Calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on another enemy party member
2. Heal another enemy party member by 5 `hp`
3. Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on another enemy party member
4. Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on another enemy party member
5. Does nothing
6. Kills themselves

Move 6 is always (and only) used if they are the last `enemydata` remaining.

For for move 1 through 4, the decision of which moves to use is complex and it involves checking every enemy party members to select a receiver and the move to use on them. Each are checked in `enemydata` order to see if one of these 4 moves can be used on them. Each moves has requirements that must be fufilled on the potential receiver and it may also require an RNG check or for the potential receiver to not be this enemy. For each receiver, each requirement and RNG check are done in move order (meaning move 1 has the highest priority per receiver while move 4 has the lowest).

Here is a table that summarises all of these requirements:

|Move|Receiver requirement|RNG check required?|Receiver can be this enemy?|
|---:|--------|--------|---------|
|1|The receiver is [stopped](../../Actors%20states/IsStopped.md) or they have the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition while their `hp` is above 0|No, the move is always used if the receiver requirement is met|Yes<sup>1</sup>|
|2|The receiver's [HPPercent](../../Actors%20states/HPPercent.md) is lower than 0.65|Yes, 50% chance to pass|Yes|
|3|The receiver doesn't already have the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition|Yes, 50% chance to pass|No|
|4|The receiver doesn't already have the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition|Yes, 50% chance to pass|No|

1: Practically, being able to cure conditions on themselves only matters if this eneny has the [Poison](../../Actors%20states/BattleCondition/Poison.md) condition because if this enemy is [stopped](../../Actors%20states/IsStopped.md), they wouldn't have been able to perform this action.

If a receiver is found, the move will be used with the receiver being the enemy party members that gets the benefit of the move.

If no receivers could be found (which typically happens if all the RNG checks applicable failed), move 5 is used.

## Move 1 - [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on another enemy party member
Calls [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) on another enemy party member. No damages are dealt.

### Logic sequence

- animstate set to 100
- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- y `spin` set to 15.0
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `spin` zeroed out
- The selected receiver's `spin` is zeroed out
- DeathSmoke particles plays at the receiver position with 2.0x size
- `Heal3` sound plays
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on the receiver
- Yield for 0.5 seconds

## Move 2 - Heal another enemy party member by 5 `hp` 
Heal another enemy party member by 5 `hp`. No damages are dealt.

### Logic sequence

- animstate set to 100
- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- y `spin` set to 15.0
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `spin` zeroed out
- The selected receiver's `spin` is zeroed out
- [Heal](../../Actors%20states/Heal.md) called to heal the receiver by 5 `hp`
- Yield for 0.5 seconds

## Move 3 - Gives [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on another enemy party member
Gives the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on another enemy party member. No damages are dealt.

### Logic sequence

- The selected receiver's y `spin` is set to 15.0
- animstate set to 100
- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- y `spin` set to 15.0
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `spin` zeroed out
- The selected receiver's `spin` is zeroed out
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [AttackUp](../../Actors%20states/BattleCondition/AttackUp.md) condition on the receiver for 3 main turns with effect
- Yield for 0.5 seconds

## Move 4 - Gives [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on another enemy party member
Gives the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on another enemy party member. No damages are dealt.

### Logic sequence

- The selected receiver's y `spin` is set to 15.0
- animstate set to 100
- `Magic` sound plays
- `MagicUp` particles plays at this enemy position
- y `spin` set to 15.0
- Yield for 0.5 seconds
- Yield for 0.25 seconds
- `spin` zeroed out
- The selected receiver's `spin` is zeroed out
- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) called to inflict the [DefenseUp](../../Actors%20states/BattleCondition/DefenseUp.md) condition on the receiver for 3 main turns with effect
- Yield for 0.5 seconds

## Move 5 - Does nothing
Does nothing. No damages are dealt

### Logic sequence

- `target` is set to a random `enemydata` index and it is rerolled if it matches this enemy or if the enemy party memebr is [stopped](../../Actors%20states/IsStopped.md). It should be noted that not only it is guaranteed that no other enemy party members is stopped by this point, but that this doesn't do anything. `target` cannot influence anything by changing it so this entire logic doesn't do anything significant

## Move 6 - Kills themselves
Kills themselves. No damages are dealt.

### Logic sequence

- Done 2 times:
    - [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move this enemy + 1.0 in x with 45.0 frametime using 101 as movestate and 103 as stopstate
    - Yield for 1 second
    - Yield for 0.25 seconds
- Yield for 0.5 seconds
- FlipSimple called on this enemy
- Yield for 0.1 seconds
- startp set to (15.0, 0.0, 0.0)
- [ForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#forcemove) called to move this enemy to startp with 60.0 frametime using 102 as movestate and stopstate
- `exp` set to 0
- `hp` set to 0 (this will cause [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md) to kill this enemy later)
- [deathtype](../../Actors%20states/Enemy%20features.md#deathtype) set to 0
- `destroytype` set to [None](../../../Entities/EntityControl/Notable%20methods/Death.md#none)
- Yield for 0.5 seconds
