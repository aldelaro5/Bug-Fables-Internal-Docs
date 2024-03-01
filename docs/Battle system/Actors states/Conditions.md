# Conditions
A condition is a temporary effect applied for a limited amount of actor turns on an actor. They are tracked in the actor's `condition` array.

A `condition` element contains an int\[\] of 2 elements:

- 0: The BattleCondition (in int form) representing the kind of effects this condition involves
- 1: The amount of actor turns left for the effects to apply. 0 and below values are technically allowed, but frequently causes the whole condition to be removed and they are not considered to be present in the actor

Some conditions are tested for resistance before being added to `condition`. This usually involves the actor's resistance compared to a random number with immunity starting at a resistance of 100 and a certainty of infliction being 0 with everything in between being (100 - resistance) / 100 chance to inflict. The presence of this test, its methodology and its accuracy (not all have the correct odds) depends on the specific method used to add the condition.

Conditions are used heavily throughout the battle system and even come with a set of utility methods to inflict / amend one, check the presence of one and remove one. Here are links to learn more about these methods:

- MainManager.[SetCondition](Conditions%20methods/SetCondition.md)
- [StatusEffect](Conditions%20methods/StatusEffect.md)
- MainManager.[HasCondition](Conditions%20methods/HasCondition.md)
- MainManager.[RemoveCondition](Conditions%20methods/RemoveCondition.md)
- [HealCondition](Conditions%20methods/HealConditions.md)
- [ClearStatus](Conditions%20methods/ClearStatus.md)

## BattleCondition table
Here are the different BattleCondition that exists in the game and a summary of what they do (with links to further documentations):

|ID|Name|Summary|
|-:|----|-------|
|0|[Freeze](BattleCondition/Freeze.md)|A stopping condition that encases the actor in a block of ice. Most attacks will remove this condition and thaw out the ice, but with a +1 damage to the attack. It also allows the target to process a `FrostBite` [medal](../../../Enums%20and%20IDs/Medal.md) when attacked|
|1|[Poison](BattleCondition/Poison.md)|A condition that either deals non lethal damage or heals the actor each actor turn depending on if they have a `ReversePoison` [medal](../../../Enums%20and%20IDs/Medal.md) or not. It also allows several poison specific medals to take effect|
|2|[Numb](BattleCondition/Numb.md)|A stopping condition that gives the actor a point of defense (with caveats, see [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `ShockTrooper` [medal](../../../Enums%20and%20IDs/Medal.md) which can bypass damage calculation|
|3|[Sleep](BattleCondition/Sleep.md)|A stopping condition that will heal the attacker on each actor turn and be removed on most attacks due to waking up (with caveats, see [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `HeavySleeper` [medal](../../../Enums%20and%20IDs/Medal.md) which can prevent waking up, halve the damage amount and heal for more per actor turn|
|4|[AttackUp](BattleCondition/AttackUp.md)|A condition that gives a +1 damage bonus on attacks by the actor while it is active when the [AttackProperty](../../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`|
|5|[DefenseUp](BattleCondition/DefenseUp.md)|A condition that gives a -1 damage on attacks to the actor as a point of defense recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#truedef) while the condition is active and property isn't `NoExceptions` (with several caveats, check the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) to learn more)|
|6|[AttackDown](BattleCondition/AttackDown.md)||
|7|[DefenseDown](BattleCondition/DefenseDown.md)||
|8|[Topple](BattleCondition/Topple.md)||
|9|[Flipped](BattleCondition/Flipped.md)||
|10|[Shield](BattleCondition/Shield.md)||
|11|[Taunted](BattleCondition/Taunted.md)||
|12|[Sturdy](BattleCondition/Sturdy.md)||
|13|[GradualHP](BattleCondition/GradualHP.md)||
|14|[GradualTP](BattleCondition/GradualTP.md)||
|15|[Eaten](BattleCondition/Eaten.md)||
|16|[EventStop](BattleCondition/EventStop.md)||
|17|[Fire](BattleCondition/Fire.md)||
|18|[Inked](BattleCondition/Inked.md)||
|19|[Sticky](BattleCondition/Sticky.md)||
|20|[Reflection](BattleCondition/Reflection.md)||
