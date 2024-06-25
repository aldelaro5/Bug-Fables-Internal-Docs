# `Sneil`

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does 1 change:

- In the projectile beam, the projectile takes 1 / 0.04 frames (25 frames) for the projectile to move to its target instead of 1 / 0.03 frames (~33.3333333334 frames)

## Move selection
1 move is possible:

1. A single target projectile beam

Move 1 is always done.

## Move 1 - Projectile beam
A single target projectile beam

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen|2|[Sleep](../../Damage%20pipeline/AttackProperty.md#attackproperty)|This enemy|`playertargetID`|A new GameObject rooted with a SpriteRenderer using the `projectilepsrites[4]` sprite (a bolt projectile)|0.03 (~33.33333334 frames of movement) if hardmode is false, 0.04 if it's true (25 frames of movement)|0.0|`keepcolor`|`deathsmokelow`|null|null|Vector3.zero|false|

### Logic sequence

- `checkingdead` is set to new coroutine called SnailBeam which will set `checkingdead` to null upon completion
- Yield all frames until `checkingdead` is null

This is what the coroutine effectively does:

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
- Camera moves to look near the midpoint between this enemy and `playertargetentity` position
- animstate set to 100
- Yield for 0.5 seconds
- animstate set to 101
- `Lazer` sound plays
- The projectile GameObject is created rooted with position being this enemy position + (-1.25, 1.0, -0.1), angles of 90.0 in z, scale of (0.5, 0.5, 0.5), sprite of `projectilepsrites[4]` sprite (a bolt projectile) and magenta color (the color is ignored because of the `keepcolor` argument sent to Projectile)
- Projectile call 1 happens
- Yield all frames until the projectile is null (it was destroyed meaning Projectile is done)
- Yield for 0.5 seconds
- `checkingdead` set to null informing the caller that the coroutine completed
