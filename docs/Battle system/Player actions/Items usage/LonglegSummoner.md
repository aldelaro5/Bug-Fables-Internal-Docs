# `LonglegSummoner`
This action has a special id being -2 and it is reserved specifically when using a `LongLegSummoner` [item](../../../Enums%20and%20IDs/Items.md).

## No charges usage (items and skill)
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic Sequence

What happens here is:

- `checkingdead` is set to a new LongLeg coroutine with the attacker as the entity and `enemydata[target]` as the target
- Yield until `checkingdead` is null (happens when LongLeg is fully done)

So most of the logic is into the LongLeg coroutine which is what will be documented here.

### LongLeg
A coroutine specific to a `LongLegSummoner` [item](../../../Enums%20and%20IDs/Items.md) item logic. entity is the user and target is the receiver of the item's effects.

It interestingly supports the target being a player party member and an entity being an enemy party member which allows to have an enemy party member simulate the usage of the item as if the player party used it during enemy actions.

It is expected that the caller sets the coroutine to `checkingdead` which will be set to null when the coroutine is done.

```cs
private IEnumerator LongLeg(EntityControl entity, MainManager.BattleData target)
```

#### [DoDamage](../../Damage%20pipeline/DoDamage.md) calls

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1_a|target is a player party member|null|`playerdata[playertargetID]`|6|[Flip](../../Damage%20pipeline/AttackProperty.md)<sup>1</sup>|empty array|`commandsuccess`|
|1_b|target is an enemy party member|null|target enemy party member|6|[Flip](../../Damage%20pipeline/AttackProperty.md)|empty array|false|

1: This property gets overriden to null in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) as the target is a player party member so it does nothing.

Here is its logic:

- `dontusecharge` set to true
- Camera moves slowly towards the target slightly to the left, up and behind
- entity animstate set to 28 (`TossItem`)
- A new sprite is created rooted a bit up from the entity using the `LonglegSummoner` [item](../../../Enums%20and%20IDs/Items.md)'s `itemsprites`
- `Toss` sound plays
- Over the course of 51.0 frames, the sprite moves towards the front of the target (relative to which side the target is) using a BeizierCurve3 with a ymax of 5.0. During the move, the z angle is set to framestep * -10.0 (* -1.0 if the target is on the left of the entity)
- entity animstate reset to the value before this coroutine
- `LongLegsGrow` sound plays
- `Prefabs/Objects/LongLegs` instantiated offscreen with Vector3.zero scale either rotated 0.0 or 180.0 in y depending on which side the target is and its animator is obtained
- DeathSmoke particles plays where the sprite landed
- The sprite is destroyed
- Yield for a frame
- `Prefabs/Objects/LongLegs` position set to where the sprite was
- Over the course of 31.0 frames, the `Prefabs/Objects/LongLegs` moves up by 4.75 in y via a SmoothLerp while its scale goes to (1.0, 1.0, -1.0) via a SmoothLerp
- The `Stomp` animation clip plays from `Prefabs/Objects/LongLegs`
- `Grow1` sound plays
- Yield for 0.75 seconds
- The `Stomp2` animation clip plays from `Prefabs/Objects/LongLegs`
- `longLegsStomp` sound plays
- Yield for 0.15 seconds
- DoDamage 1_a or 1_b happens depending on the party of target
- Yield for 0.75 seconds
- Over the course of 31.0 frames, the `Prefabs/Objects/LongLegs` moves down by 4.75 in y via a SmoothLerp while its scale goes to Vector3.zero via a SmoothLerp
- DeathSmoke particles plays at the `Prefabs/Objects/LongLegs` position
- `ChargeDown2` sound plays
- `Prefabs/Objects/LongLegs` is destroyed
- Yield for 0.5 seconds
- `checkingdead` is set to null which signals the caller that the coroutine is done
