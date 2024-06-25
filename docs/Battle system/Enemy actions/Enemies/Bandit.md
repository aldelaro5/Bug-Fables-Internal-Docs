# `Bandit`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true has 1 change:

- In the projectile throw, the projectile takes 1 / 0.045 frames (~22.22222223 frames) for the projectile to move to its target instead of 1 / 0.025 frames (40.0 frames)

## Move selection
3 moves are possible:

1. A single target tackle attack that may steal an [item](../../../Enums%20and%20IDs/Items.md)
2. A single target projectile throw
3. Flees the battle

Move 3 is only used if this enemy has an `holditem` and it is the only one that will be used under this condition.

Otherwise, move 1 and 2 is chosen with these odds:

- Move 1: 4/10
- Move 2: 6/10

## Move 1 - Tackle attack
A single target tackle attack that may steal an [item](../../../Enums%20and%20IDs/Items.md).

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- animstate set to 101
- Yield for 0.1 seconds
- `Gleam` particles played with `Gleam` sound near this enemy with an alivetime of 0.5
- Yield for 0.5 seconds
- `trail` set to true
- animstate set to 1 (`Walk`)
- `Scuttle` sound plays on loop using `sounds[9]`
- Over the course of 25.0 frames position lerped from startp to `playertargetentity` position via a BeizierCurve3 with a ymax of the midpoint between startp and `playertargetentity` + -3.0 in z
- Once the 25.0 frames are done:
    - `flip` set to true
    - DoDamage 1 call happens
    - If `playerdata[playertargetID]` doesn't have the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition, [StealItem](../StealItem.md) is called
    - If an [item](../../../Enums%20and%20IDs/Items.md) was stolen from the call above, the local startstate is set to 4 (`ItemGet`)
- Over the course of 25.0 frames position lerped from `playertargetentity` position to startp via a BeizierCurve3 with a ymax of the midpoint between startp and `playertargetentity` + 3.0 in z
- `sounds[9]` is stopped
- position set to startp
- `flip` set to false
- `trail` set to false

## Move 2 - Projectile throw
A single target projectile throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on any player party member whose `hp` is above 0.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen|2|null|This enemy|`playertargetID`|A new GameObject named `proj` rooted with a SpriteRenderer using the `projectilepsrites[3]` sprite (a triwing projectile)|0.025 (40.0 frames of movement) if hardmode is false, 0.045 if it's true (~22.222223 frames of movement)|0.0|null|null|null|null|(0.0, 0.0, 20.0)|false|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity`
- A new GameObject named `proj` is created with a SpriteRenderer rooted
- animstate set to 100
- Yield for 0.5 seconds
- `proj` position set to this enemy + (0.75, 2.0, -0.1)
- `proj` sprite set to `projectilepsrites[3]` sprite (a triwing projectile)
- `spin3` sound plays on loop using `sounds[9]`
- `Toss` sound plays
- Projectile 1 call happens
- Yield all frames until `proj` is null (it was destroyed meaning Projectile is done)
- `sounds[9]` stopped
- Yield for 0.6 seconds

## Move 3 - Flee
Flees the battle, no damages are dealt.

### Logic sequence

- `tryenemyheal` set to a new [EnemyFlee](../EnemyFlee.md) coroutine with walkstate 27 (`ItemWalk`) with afterimage on this enemy
- Yield all frames until `tryenemyheal` goes to null (the coroutine completed)
- The local fled is set to true indicating a different [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#fled-enemy-post-action) will be done
