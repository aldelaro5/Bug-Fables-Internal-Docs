# ActionBehaviors
An `ActionBehaviors` is an enum type that represents a behavior which has some logic to perform on the active main section of Update. It only applies to all [NPCType](NPCType.md) except [Object](Object.md).

There are 2 `ActionBehaviors` defined in [NPCControl data](../../TextAsset%20Data/Entity%20data.md#map-entity-data) with the first one being the default one and the second one being only used when `inrange` is true. Each comes with an `actionfrequency` field that has a different meaning depending on the behavior.

The actual execution of the behavior is done by [DoBehavior](Notable%20methods/DoBehaviour.md) called by [Update](Update.md) which receives the appropriate frequency, but only on the main active section. This means that it is only called when the NPCControl is considered active and a main update occurs (which excludes the special cases of an [Enemy](Enemy.md) when it is dizzy, about to be unfrozen or frozen).

## Enum table
The following is a table of the different enum values (NOT REFERENCED means the game never reference the value meaning it has not logic backing it, UNUSED means the behavior has logic, but it is either dead or not practically used under normal gameplay):
|Value|Name|Description|
|----:|----|----------|
|0|[None](ActionBehaviors/None.md)| |
|1|[FacePlayer](ActionBehaviors/FacePlayer.md)| |
|2|[ChasePlayer](ActionBehaviors/ChasePlayer.md)| |
|3|[FleeFromPlayer](ActionBehaviors/FleeFromPlayer.md)| |
|4|[TurnRandomly](ActionBehaviors/TurnRandomly.md)| |
|5|[Wander](ActionBehaviors/Wander.md)| |
|6|[FaceAwayFromPlayer](ActionBehaviors/FaceAwayFromPlayer.md)| |
|7|[TurnFixedInterval](ActionBehaviors/TurnFixedInterval.md)| |
|8|[Disguise](ActionBehaviors/Disguise.md)| |
|9|[DisguiseOnce](ActionBehaviors/DisguiseOnce.md)| |
|10|FollowPlayer|NOT REFERENCED|
|11|[WalkAwayFromPlayer](ActionBehaviors/WalkAwayFromPlayer.md)| |
|12|[FaceAhead](ActionBehaviors/FaceAhead.md)| |
|13|[FaceBehind](ActionBehaviors/FaceBehind.md)| |
|14|[FaceUp](ActionBehaviors/FaceUp.md)| |
|15|[FaceDown](ActionBehaviors/FaceDown.md)| |
|16|[SetPath](ActionBehaviors/SetPath.md)| |
|17|[ChargeAtPlayer](ActionBehaviors/ChargeAtPlayer.md)| |
|18|[ChargeAtPlayerFlipSprite](ActionBehaviors/ChargeAtPlayerFlipSprite.md)| |
|19|[ShootProjectile](ActionBehaviors/ShootProjectile.md)| |
|20|[ShootProjectilePredict](ActionBehaviors/ShootProjectilePredict.md)| |
|21|[ChargeAndAttack](ActionBehaviors/ChargeAndAttack.md)| |
|22|[AlwaysWander](ActionBehaviors/AlwaysWander.md)| |
|23|[Unmoveable](ActionBehaviors/Unmoveable.md)| |
|24|[ChargeAttackUnderground](ActionBehaviors/ChargeAttackUnderground.md)| |
|25|[WanderUnderground](ActionBehaviors/WanderUnderground.md)| |
|26|[StealthAI](ActionBehaviors/StealthAI.md)| |
|27|[SetPathJump](ActionBehaviors/SetPathJump.md)| |
|28|[DisguiseOnceJumpForward](ActionBehaviors/DisguiseOnceJumpForward.md)|UNUSED, An alias of `Disguise`|
|29|[ChangeSpriteInRandius](ActionBehaviors/ChangeSpriteInRandius.md)| |
|30|[ChaseWhenAnim](ActionBehaviors/ChaseWhenAnim.md)| |
|31|[WalkWhenAnim](ActionBehaviors/WalkWhenAnim.md)| |
|32|[WanderOffscreen](ActionBehaviors/WanderOffscreen.md)| |
|33|[WanderNoWarp](ActionBehaviors/WanderNoWarp.md)| |
|34|[WanderOnWater](ActionBehaviors/WanderOnWater.md)| |
|35|[ChaseOnWater](ActionBehaviors/ChaseOnWater.md)| |