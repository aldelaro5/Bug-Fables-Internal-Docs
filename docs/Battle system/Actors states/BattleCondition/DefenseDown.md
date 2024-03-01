# DefenseDown
A condition that removes a point of defense recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#truedef) which can result in a +1 damage effects while it is active when the [AttackProperty](../../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`. NOTE: This has several caveats, check the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) to learn more.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition, the turn amount is always increased by the existing one (meaning it stacks).

When inflicted as a new condition, nothing special happens.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it is always inflicted to the actor when called.

It includes optional visual effects when effect is true:

- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with type 3 (blue down arrow)
- The `StatDown` sound is played if it wasn't already

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
If the attacker has this condition, a defense may be removed resulting in a -1 damage effect that applies unless property is `NoExceptions`. NOTE: This has several caveats, check the method's documentation to learn more. Notably, it prevents the [Numb](Numb.md) condition to add a defense.

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
After damage calculations, if the target's `hp` is still above 0 and it doesn't have the [Shield](Shield.md) condition, this condition will be inflicted via [StatusEffect](../Conditions%20methods/StatusEffect.md) if property is `DefDownOnBlock`. The amount of turn to inflict is 2 turns unless block is true where it's 1 turn instead. On top of this, it inflicts for one more turn if the target is a player party member (meaning 3 turns or 2 with block).
