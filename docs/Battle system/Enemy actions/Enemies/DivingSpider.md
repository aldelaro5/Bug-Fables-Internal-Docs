# `DivingSpider`

## `data` usage
At the start of the action, if `data` is null or empty, it's initialised to be 1 element with a starting value of 0.

- `data[0]`: An actor turn cooldown for inflicting themselves the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition in the post move logic. When inflicting themselves the condition in the post move logic, the value is set to 2. It is decremented at the end of the action if it's not 0, but the decrement is at the expense of not being able to inflict themselves the condition so it prevents it. It's only when the value becomes 0 (after 2 completed actor turns) that this enemy can inflict themsevles with the `Shield` condition again in the post move logic.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the water bubble projectiles throw move, the amount of hits changes to be random between 2 and 3 inclusive instead of being random between 1 and 2 inclusive
- In the water bubble projectiles throw move, speed of the Projectile call changes to be 27.0 (35.0 if launched with higher height) instead of being 35.0 (42.0 if launched with higher height)

## Move selection
1 moves is possible:

1. A multiple targets water bubble projectiles throw

Move 1 is always used.

## Pre move logic
The following logic always happen before using a move:

- If `hologram` is true and `sprite`'s renderQueue isn't 2500:
    - `sprite`'s renderQueue is set to 2500
    - All Renderer childed to `extraanims[0]`'s parent's parent (this should be the same as the `model`) has their renderQueue set to 2500

## Post move logic
The following logic is always done after using a move:

- If `data[0]` is 0 (the cooldown on the shield expired):
    - If a 5/10 RNG check passes:
        - `Charge19` sound plays
        - animstate set to 100
        - y `spin` set to 15.0
        - Yield for 0.5 seconds
        - CreateShield called on this enemy which initialises their `bubbleshield`
        - `Shield` sound plays
        - `bubbleshield`'s DialogueAnim's `targetscale` is set to (3.0, 3.0, 2.0)
        - `overrideshieldpos` set to (0.25, 1.0, 0.0)
        - `shieldenabled` set to true
        - [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) called to inflict the [Shield](../../Actors%20states/BattleCondition/Shield.md) condition on this enemy for 2 main turns (effectively 1 main turn as the current main turn ends soon after)
        - `data[0]` set to 2 (meaning this enemy won't be able to get inflicted this condition again until they completed 2 full actor turns)
        - Yield for 0.5 seconds
        - `spin` zeroed out
- Otherwise (the cooldown hasn't expired yet):
    - `data[0]` is decremented

## Move 1 - Water bubbles projectiles throw
A multiple targets water bubble projectiles throw.

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [Projectile](../../Damage%20pipeline/Projectile.md) calls

|#|Conditions|damage|property|attacker|playertarget|obj|speed|height|extraargs|destroyparticle|audioonhit|audiomoving|spin|nosound|
|-:|---------|------|--------|--------|-----------|---|-----|------|---------|--------------|----------|-----------|----|------|
|1|Always happen from 1 to 2 times (2 to 3 times instead if hardmode is true), but the call and further calls won't happen if there's not at least 1 player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences))|2|null or [Poison](../../Damage%20pipeline/AttackProperty.md) determined randomly|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) (target changes for each calls)|A new sprite object rooted using the `Sprites/Particles/waterbubble` sprite positioned at this enemy + (-1.0, 0.75, -0.1) with a scale of 0.35x and a pure magenta color if propert is `Poison`|One of the 2 chosen randomly with uniform odds check:<ul><li>35.0 (27.0 instead if hardmode is true)</li><li>42.0 (35.0 instead if hardmode is true)</li></ul>|0.0 if using the faster speed, 3.0 if using the slower speed|`keepcolor`|`WaterSplash` (`PoisonEffect` instead if property is `Poison`)|`BubbleBurst`|null|Vector3.zero|false|

### Logic sequence

- The amount of hits is determined. It's random between 1 and 2 inclusive (random between 2 and 3 inclusive instead if hardmode is true). An array of SpriteRenderer is initialised with this amount as the length to hold the projectiles objects
- For each hit to do, if there's still at least one player party member alive (`hp` above 0 and not [eatenby](../../Actors%20states/BattleCondition/Eaten.md#eatenby-influences)):
    - [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) called
    - Camera moves to look near the midpoint between this enemy and `playertargetentity`
    - `Blosh` sound plays
    - animstate set to 100
    - If a 5/10 RNG check passes, the hit will be done with slower speed / higher height and ShakeSprite is called on this enemy for 0.1 intensity and 30.0 frametimer
    - Yield for 0.5 seconds
    - The projectile element is set to a new sprite object rooted using the `Sprites/Particles/waterbubble` sprite positioned at this enemy + (-1.0, 0.75, -0.1) with a scale of 0.35x and a pure magenta color if propert is `Poison` (which is determined by a 5/10 RNG check)
    - `Toss16` sound plays
    - Projectile 1 call happens
    - animstate set to 101
    - MainManager.MoveTowards called to move this enemy from its position + 1.0 in x to its position with 10.0 frametime with smooth without caller (done in paralel)
    - Yield for 0.5 seconds
- animstate set to 0 (`Idle`)
- Yield all frames until all projectiles elements are null (all Projectile 1 calls completed)
- Yield for 0.5 seconds
