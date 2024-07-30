# `Tablet`

## Assumptions
It is assumed that a [EverlastingKing](EverlastingKing.md) is always present alongside this enemy as otherwise, this enemy will cause an exception to be thrown.

## [Fire](../../Actors%20states/BattleCondition/Fire.md) damage infliction logic in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This enemy cannot be on fire meaning any `Fire` property damages will not inflict the [Fire](../../Actors%20states/BattleCondition/Fire.md) condition.

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: An actor cooldown before using the shield move. When the move is used, the value is set to 2 (1 instead if hardmode is true). Every actor turn that this value is above 0, it is decremented, but at the expense of not using the shield move meaning it prevents it. The move only becomes usable again when the value is 0 or lower. Effectively, it's an antispam of 2 (1 if hardmode is true) completed actor turns before the shield move can be used

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The value `data[0]` is set to when using the shield move changes to 1 from 2. It means that this enemy only needs to wait for 1 complete actor turn before using the move again instead of having to wait 2 complete actor turns

## Move selection
1 moves is possible:

1. Places a shield on [EverlastingKing](EverlastingKing.md) which inflicts them the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition

Move 1 is always (and only) used if `data[0]` is 0 or below (the cooldown on the usage of the move expired).

Otherwise, the action is limited to `data[0]` being decremented, but move 1 won't be used.

## Move 1 - Shields [EverlastingKing](EverlastingKing.md)
Places a shield on [EverlastingKing](EverlastingKing.md) which inflicts them the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition. No damages are dealt.

### Logic sequence

- y `spin` set to 20.0
- `EverLastingKingSummonArtifact` sound plays
- `TabletSpawn` particles plays at this enemy position + 2.0 in y with 0.5 alivetime
- Yield for 0.5 seconds
- The first occurence of [EverlastingKing](EverlastingKing.md) in `enemydata` is obtained (its presence is assumed)
- CreateShield called on `EverlastingKing`
- `Shield` sound plays
- `EverlastingKing`'s `bubbleshield`'s `targetscale` set to (1.5, 3.0, 2.0)
- `EverlastingKing`'s `overrideshieldpos`'s set to (0.0, 1.65, 0.0)
- `EverlastingKing`'s `shieldenabled`'s set to true
- [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on `EverlastingKing` for 2 main turns
- `data[0]` set to 2 (1 instead if hardmode is true)
- SlowSpinStop called on this enemy's `spin` with 30.0 frametime
- Yield for 0.5 seconds
