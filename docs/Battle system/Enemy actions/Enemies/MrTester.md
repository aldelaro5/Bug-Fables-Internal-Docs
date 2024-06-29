# `MrTester`

## About this enemy
This isn't an enemy meant to be encountered under normal gameplay, but it does exist and has an action implementation. However, this implementation just heals their `hp` and the player party's `tp` with no way to attack. Since it is meant for this enemy to have high `hp`, it is effectively not defeatable normally.

The purpose of such an enemy is for testing purposes only since it allows to attack someone without having to restart a battle due to the enemy dying.

## Move selection
1 move is possible:

1. Heals their `hp` to their `maxhp` as well as the player party's `tp` to their `maxtp`

Move 1 is always done

## Move 1 - Healing
Heals their `hp` to their `maxhp` as well as the player party's `tp` to their `maxtp`. No damages are dealt

### Logic sequence

- `hp` set to `maxhp`
- HealParticle plays at this enemy with a size of Vector3.one and an offset of Vector3.up
- `Heal` sound plays
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 1 (HP counter), an ammount of this enemy's `maxhp` starting at this enemy position + 2.0 in y and ending at this enemy position + 5.0 in y
- Yield for 0.5 seconds
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) called with type 2 (TP counter), an ammount of 99 starting at the front player party member + 2.0 in y and ending at the front player party member + 5.0 in y 
- instance.`tp` set to instance.`maxtp`
- Yield for 0.5 seconds
