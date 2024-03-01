# DefenseUp
A condition that gives a -1 damage on attacks to the actor as a point of defense recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#truedef) while the condition is active and property isn't `NoExceptions` (with several caveats, check the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) to learn more).

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition, the turn amount is always increased by the existing one (meaning it stacks).

When inflicted as a new condition, nothing special happens.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it is always inflicted to the actor when called.

It includes optional visual effects when effect is true:

- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with type 1 (blur up arrow)
- The `StatUp` sound is played if it wasn't already

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
If the attacker has this condition, a -1 damage effect is applied unless property is `NoExceptions` / `Flip` or piercing applies. NOTE: This has several caveats, check the method's documentation to learn more (including incorrect logic concerning this condition and the `AntlionJaws` [medal](../../../Enums%20and%20IDs/Medal.md)).

## [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)
When summoning `extraenemies` at the `AbandonedCity` [map](../../../Enums%20and%20IDs/Maps.md) while [flags](../../../Flags%20arrays/flags.md) 400 is true, this condition is inflicted to all enemies alongside [AttackUp](AttackUp.md) for 999999 turns.
