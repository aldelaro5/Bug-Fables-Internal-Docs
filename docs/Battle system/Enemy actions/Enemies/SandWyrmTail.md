# `SandWyrmTail`

## Special enemy party layout
See the [SandWyrm](SandWyrm.md) documentation to learn more about how it interacts with this enemy very closely together. This enemy can be viewed as an instrument of the `SandWyrm`.

## Assumptions
It is assumed that this enemy has an `eventondeath` set to the proper [EventDialogue](../../Battle%20flow/EventDialogues/SandWyrm%20and%20SandWyrmTail.md#sandwyrm-and-sandwyrmtail-eventdialogues) because this EventDialogue needs to inflict [Flipped](../../Actors%20states/BattleCondition/Flipped.md) on the [SandWyrm](SandWyrm.md).

## [StartBattle](../../StartBattle.md) special logic
This enemy features special logic during StartBattle before and after it is loaded.

### Before the enemy is loaded ([EnemyCheck](../../StartBattle%20phases/Pre%20haltbattleload.md#enemycheck))
If StartBattle was called with any elements of enemyids containing this enemy or [SandWyrm](SandWyrm.md), the entire enemyids is overriden to be {[SandWyrm](SandWyrm.md), `SandWyrmTail`}.

### After the enemy is loaded ([enemy party setup](../../StartBattle%20phases/Post%20haltbattleload.md#enemy-party-setup))
If `enemydata` contains this enemy, a VenusBattle is added to the first 2 elements of the enemy party where the parent is the other enemy and the target is themselves. Since the EnemyCheck logic already enforced the layout of the enemy party, it effectively means that:

- This enemy gets a VenusBattle SetUp with a target of themselves and the parent being the [SandWyrm](SandWyrm.md)
- The [SandWyrm](SandWyrm.md) gets a VenusBattle SetUp with a target of themselves and the parent being this enemy

This means that each have a VenusBattle that checks each other to make sure they have matching animstate and animations. It allows to keep them in sync visually in a consistent manner.

This part of the logic also changes the `battlepos` of this enemy and the [SandWyrmTail](SandWyrmTail.md):

- The [SandWyrm](SandWyrm.md): (3.0, 0.0, 0.0)
- This enemy: (5.9, 0.0, 0.0)

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- In the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) move, the amount of damage the projectile will do changes to 3 from 2
- In the [delayed projectile](../../Actors%20states/Delayed%20projectile.md) move, the framespeed changes to 45.0 from 55.0

## Move selection
1 moves is possible:

1. Launches a bubble [delayed projectile](../../Actors%20states/Delayed%20projectile.md)

Move 1 is always used, but only if the [SandWyrm](SandWyrm.md) is not [stopped](../../Actors%20states/IsStopped.md) (it should always be present due to the EnemyCheck logic).

## Move 1 - [delayed projectile](../../Actors%20states/Delayed%20projectile.md) launch
Launches a bubble [delayed projectile](../../Actors%20states/Delayed%20projectile.md). No damages are dealt.

### [AddDelayedProjectile](../../Actors%20states/Delayed%20projectile.md#adddelayedprojectile) calls

|#|Conditions|obj|targetpos|damage|turnstohit|areadamage|property|framespeed|summonedby|hitsound|hitparticle|whilesound|
|-:|---------|---|---------|------|---------|----------|--------|----------|----------|--------|----------|----------|
|1|Always happen|A new `Prefabs/Objects/WaterBubble` GameObject positioned at (0.0, 0.15, 0.0) childed to the `battlemap` with a scale of 1.5x|[GetRandomAvaliablePlayer](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) without nullable|2 (3 instead if hardmode is true)|2|0|null|55.0 (45.0 instead if hardmode is true)|This enemy|`WaterSplash2`|`WaterSplash`|`Fall2`|

### Logic sequence

- The [SandWyrm](SandWyrm.md) enemy party member is obtained (it should always be present due to the logic enforced by EnemyCheck)
- Camera moves to look near the midpoint between this enemy and the `SandWyrm`, but a bit up
- `SandWyrm` animstate set to 100
- animstate set to 100
- `TidalWormDropplet` sound plays
- Yield for 0.5 seconds
- A new `Prefabs/Objects/WaterBubble` GameObject is created positioned at this enemy + (0.5, 0.0, -0.1) childed to the `battlemap`
- A new `Prefabs/Objects/WaterSplash` GameObject is created childed to the `Prefabs/Objects/WaterBubble` with its AudioSource disabled and destroyed in 0.75 seconds
- The `Prefabs/Objects/WaterBubble` scale is set to 1.5x
- `WaterOut` sound plays
- Over the course of 61.0 frames, `Prefabs/Objects/WaterBubble` moves to (0.0, 15.0, 0.0) via a lerp
- `SandWyrm` animstate set to 0 (`Idle`)
- AddDelayedProjectile call 1 happens
