# `Mushroom`

## randomposafter set to true
This enemy action always sets the randomposafter local to true. This means that in [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#no-fled-enemy-post-action) when no enemy fled, this enemy's [position](../../Actors%20states/BattlePosition.md) will be randomly determined between `Ground` and `Flying`.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes
HardMode being true does the following changes:

- The move selection odds changes to be uniform odds to use any moves instead of being biased towards the aerial spin attack

## Move selection
2 moves are possible:

1. A single target aerial spin attack
2. A single target poison bubble throw

The decision of which move to use depends on odds, but the odds changes if hardmode is true. Here are the odds in either situations:

|Move|Odds when hardmode is false|Odds when hardmode is true|
|---:|---------|------|
|1|67%|33%|
|2|50%|50%|

## Move 1 - Aerial spin
A single target aerial spin attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- `Fly` sound played on loop using `sounds[8]` with 0.5 pitch
- Camera moves to look near `playerdata[playertargetID]`
- [position](../../Actors%20states/BattlePosition.md) changes to `Flying`
- `height` set to 0.125
- `Idlef` animation clip plays on `anim`
- `flyinganim` set to true
- SetAnimForce called
- Over the course of a maximum of 60.0 frames, this enemy moves to `playerdata[playertargetID]` position + (3.0, 3.25, 0.3333334) via lerp with a factor of 0.05 the game's frametime. This stops on the first frame this enemy is at a distance of 0.3 or below from the target position. Before each frame yield, the `Idlef` animation clip plays on `anim`
- x and z angles of `sprite` set to 0.0
- animstate set to 101
- `Woosh2` sound plays
- `sounds[8]` stopped
- Yield all frames until this enemy y position reaches `playerdata[playertargetID]`'s y position + 2.15 or below. On each frame, this enemy y position decreases by 0.05 the game's frametime
- animstate set to 100
- y `spin` set to 25.0
- `Spin` sound plays on loop using `sounds[9]` at 0.8 pitch and 0.6 volume
- Yield all frames until this enemy x position reaches `playerdata[playertargetID]`'s x position - 2.3 or below. On each frame, this enemy moves to `playerdata[playertargetID]` position + (2.0, 0.4, 0.0) via a lerp with a factor of 0.05 of the game's frametime. On the first frame this enemy x position becomes lower than `playerdata[playertargetID]` x position - 0.1, DoDamage 1 call happens
- `sounds[9]` stops
- `height` set to 2.0
- Yield for 0.1 seconds
- `sprite` angles reset to Vector3.zero

## Move 2 - Poison bubble throw
A single target poison bubble throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../../../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|`playertargetID` after [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget)|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- Yield all frames until `height` becomes 0.1 or below. On each frame:
    - animstate set to 3 (`Fall`)
    - `height` decreases by 0.07 * the game's frametime
- [position](../../Actors%20states/BattlePosition.md) changed to `Ground`
- Camera moves to look at this enemy, but zoomed in
- `height` set to 0.0
- animstate set to 102
- `Charge` sound plays with 0.8 pitch
- Yield for 1.25 seconds
- animstate set to 104
- `Prefabs/Objects/PoisonBubble` instantiated at this enemy position + 2.0 in y
- [SeDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) called
- `Blosh` sound plays
- `Wub` sound plays on loop using `sounds[9]` with 0.7 pitch
- Over the course of 60.0 frames the `Prefabs/Objects/PoisonBubble` moves to `playerdata[playertargetID]` position + 1.0 in y via a BeizierCurve3 with a ymax of 5.0
- `sounds[9]` stopped
- `BubbleBurst` sound plays
- `PoisonEffect` particles plays at the `playertargetentity` position
- DoDamage 1 call happens
- `Prefabs/Objects/PoisonBubble` destroyed
- Yield for 0.75 seconds
- Yield for 0.1 seconds
- `sprite` angles reset to Vector3.zero
