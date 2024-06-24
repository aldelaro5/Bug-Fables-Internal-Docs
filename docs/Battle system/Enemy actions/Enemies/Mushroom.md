# `Mushroom`

## randomposafter set to true
This enemy action always sets the randomposafter local to true. This means that in [post-action](../../Battle%20flow/Action%20coroutines/DoAction.md#no-fled-enemy-post-action) when no enemy fled, this enemy's [position](../../Actors%20states/BattlePosition.md) will be randomly determined between `Ground` and `Flying`.

## [HardMode](../../Damage%20pipeline/HardMode.md) changes changes
HardMode being true changes the odds in move selection to be unbiased to use any moves instead of being biased towards the aerial spin attack.

## Move selection
2 moves are possible:

1. A single target aerial spin attack
2. A single target poison bubble throw

The odds of which one is used depends on [HardMode](../../Damage%20pipeline/HardMode.md).

With hardmode true:

- 50% to be move 1
- 50% to be move 2

With hardmode false:

- 67% to be move 1
- 33% to be move 2

## Move 1 - aerial spin
A single target aerial spin attack

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|null|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- `Fly` sound played on loop using `sounds[8]` with 0.5 pitch
- [position](../../Actors%20states/BattlePosition.md) changes to `Flying`
- `height` set to 0.125
- `Idlef` animation clip played
- Yield up to 60.0 frames so that the position changes via a lerp until it reaches the target + (3.0, 3.25, 1.0 / 3.0) at a distance of 0.3 or below at a rate of 0.05 of the game's frametime while playing the `Idlef` animation clip
- animstate set to 101
- `Woosh2` sound plays
- `sounds[8]` stopped
- Yield frames so that the y position decreases until it reached the target's y position + 2.15
- animstate set to 100
- y `spin` set to 25.0
- `Spin` sound plays on loop using `sounds[9]` at 0.8 pitch and 0.6 volume
- Yield frames so that the position changes via a lerp to the target + (-2.0, 0.4, 0.0) until the x position reaches the target's x position - 2.3 with a factor of 0.05 of the game's frametime:
    - During the movement, if the x position gets lower than the target's x position + 0.1 for the first time, DoDamage 1 call happens
- `sounds[9]` stops
- `height` set to 2.0
- Yield for 0.1 seconds
- angles reset to Vector3.zero

## Move 2 - poison bubble throw
A single target poison bubble throw

### `nonphyscal` set to true
This move always sets `nonphyscal` to true which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md) if equipped on the target.

### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|Always happen|This enemy|The selected `playertargetID`|2|[Poison](../../Damage%20pipeline/AttackProperty.md)|null|`commandsuccess`|

### Logic sequence

- [GetSingleTarget](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md) called
- Yield all frames so that `height` decreases to 0.1 or below at a rate of 0.07 the game's frametime while using animstate 3 (`Fall`)
- [position](../../Actors%20states/BattlePosition.md) changed to `Ground`
- `height` set to 0.0
- animstate set to 102
- `Charge` sound plays with 0.8 pitch
- Yield for 1.25 seconds
- animstate set to 104
- `Prefabs/Objects/PoisonBubble` instantiated at the enemy + (0.0, 2.0, 0.0)
- `Blosh` sound plays
- `Wub` sound plays on loop using `sounds[9]` with 0.7 pitch
- Yield for 60.0 frames so that the `Prefabs/Objects/PoisonBubble` moves towards the target + (0.0, 1.0, 0.0) using a BeizierCurve3 with a ymax of 5.0
- `sounds[9]` stopped
- `BubbleBurst` sound plays
- `PoisonEffect` particles plays at the target
- `nonphyscal` set to true
- DoDamage 1 call happens
- `Prefabs/Objects/PoisonBubble` destroyed
- Yield for 0.75 seconds
- Yield for 0.1 seconds
- angles reset to Vector3.zero
