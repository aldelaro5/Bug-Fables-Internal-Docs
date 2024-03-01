# AttackDown
A condition that gives a -1 damage malus on attacks by the actor while it is active when the [AttackProperty](../../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition, the turn amount is always increased by the existing one (meaning it stacks).

When inflicted as a new condition, nothing special happens.

## [StatusEffect](../Conditions%20methods/StatusEffect.md)
This condition is supported by this method and it is always inflicted to the actor when called.

It includes optional visual effects when effect is true:

- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with type 2 (red down arrow)
- The `StatDown` sound is played if it wasn't already

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
If the attacker has this condition, a -1 damage malus is applied unless property is `NoExceptions`.

A variation of this logic is also enforced outside of the damage pipeline in GetMultiDamage. That method is used by the following [skills](../../../Enums%20and%20IDs/Skills.md):

- `IceSphere`
- `IceDrill`
- `BeeFly`

## [DoDamage](../../Damage%20pipeline/DoDamage.md)
After damage calculations, if the target's `hp` is still above 0 and it doesn't have the [Shield](Shield.md) condition, this condition will be inflicted via [StatusEffect](../Conditions%20methods/StatusEffect.md) if property is `AtkDownOnBlock`. The amount of turn to inflict is 2 turns unless block is true where it's 1 turn instead. On top of this, it inflicts for one more turn if the target is a player party member (meaning 3 turns or 2 with block).

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When an [AttackUp](AttackUp.md) condition is processed (when `hp` is above 0), if the actor's `atkdownonloseatkup` is true, this condition is inflicted by doing the following:

- actor's `atkdownonloseatkup` is set to false
- [SetCondition](../Conditions%20methods/SetCondition.md) is called to inflict this condition for 2 turns on the actor
- The `StatDown` sound is played
- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with type 2 (red down arrow)

## [UseCharm](../../Battle%20flow/UseCharm.md)
This condition prevents a charm of type `AttackUp` to take effect as its effect would become redundant. Check the method's documentation to learn more.

## [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md)
When summoning `extraenemies` at the `AbandonedCity` [map](../../../Enums%20and%20IDs/Maps.md) while [flags](../../../Flags%20arrays/flags.md) 400 is true, this condition is inflicted to all enemies alongside [DefenseUp](DefenseUp.md) for 999999 turns.
