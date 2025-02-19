# Conditions
A condition is a temporary effect applied for a limited amount of main turns on an actor. They are tracked in the actor's `condition` array.

A `condition` element contains an int\[\] of 2 elements:

- 0: The BattleCondition (in int form) representing the kind of effects this condition involves
- 1: The amount of main turns left for the effects to apply. 0 and below values are technically allowed, but frequently causes the whole condition to be removed and they are not considered to be present in the actor

It is considered invalid to have duplicate conditions in the list. An existing condition should be amended if it exist already instead of adding a new element to the list.

Some conditions are tested for resistance before being added to `condition`. This usually involves the actor's resistance compared to a random number with immunity starting at a resistance of 100 and a certainty of infliction being 0 with everything in between being (100 - resistance) / 100 chance to inflict. The presence of this test, its methodology and its accuracy (not all have the correct odds) depends on the specific method used to add the condition.

Conditions are used heavily throughout the battle system and even come with a set of utility methods to inflict / amend one, check the presence of one and remove one. Here are links to learn more about these methods:

- MainManager.[SetCondition](Conditions%20methods/SetCondition.md)
- [StatusEffect](Conditions%20methods/StatusEffect.md)
- MainManager.[HasCondition](Conditions%20methods/HasCondition.md)
- MainManager.[RemoveCondition](Conditions%20methods/RemoveCondition.md)
- [HealCondition](Conditions%20methods/HealConditions.md)
- [ClearStatus](Conditions%20methods/ClearStatus.md)
- [TryConditions](Conditions%20methods/TryConditions.md)

## BattleCondition table
Here are the different BattleCondition that exists in the game and a summary of what they do (with links to further documentations):

|ID|Name|Summary|
|-:|----|-------|
|0|[Freeze](BattleCondition/Freeze.md)|A stopping condition that encases the actor in a block of ice. Most attacks will remove this condition and thaw out the ice, but with a +1 damage to the attack. It also allows the target to process a `FrostBite` [medal](../../Enums%20and%20IDs/Medal.md) when attacked|
|1|[Poison](BattleCondition/Poison.md)|A condition that either deals non lethal damage or heals the actor each main turn depending on if they have a `ReversePoison` [medal](../../Enums%20and%20IDs/Medal.md) or not. It also allows several poison specific medals to take effect|
|2|[Numb](BattleCondition/Numb.md)|A stopping condition that gives the actor a point of defense (with caveats, see [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `ShockTrooper` [medal](../../Enums%20and%20IDs/Medal.md) which can bypass damage calculation|
|3|[Sleep](BattleCondition/Sleep.md)|A stopping condition that will heal the actor (or damage them if they are an enemy party member while the `BadDream` [medal](../../Enums%20and%20IDs/Medal.md) is equipped) on each main turn and be removed on most attacks due to waking up (with caveats, see [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation for details). It also allows the target to process a `HeavySleeper` [medal](../../Enums%20and%20IDs/Medal.md) which can prevent waking up, halve the damage amount and heal for more per main turn|
|4|[AttackUp](BattleCondition/AttackUp.md)|A condition that gives a +1 damage bonus on attacks by the actor while it is active when the [AttackProperty](../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`|
|5|[DefenseUp](BattleCondition/DefenseUp.md)|A condition that gives a -1 damage on attacks to the actor as a point of defense recognised by [TrueDef](../Visual%20rendering/RefreshEnemyHP.md#truedef) while the condition is active and property isn't `NoExceptions` NOTE: This has several caveats, check the [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation to learn more|
|6|[AttackDown](BattleCondition/AttackDown.md)|A condition that gives a -1 damage malus on attacks by the actor while it is active when the [AttackProperty](../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`|
|7|[DefenseDown](BattleCondition/DefenseDown.md)|A condition that removes a point of defense recognised by [TrueDef](../Visual%20rendering/RefreshEnemyHP.md#truedef) which can result in a +1 damage effects while it is active when the [AttackProperty](../Damage%20pipeline/AttackProperty.md) isn't `NoExceptions`. NOTE: This has several caveats, check the [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation to learn more|
|8|[Topple](BattleCondition/Topple.md)|A 1 hit defensive mechanism in the form of a condition for an enemy party member where if it has a `ToppleFirst` or `ToppleAirOnly` (if their `position` is `Flying`) [weakness](../Damage%20pipeline/AttackProperty.md) it won't get dropped or [Flipped](BattleCondition/Flipped.md) immediately when it was supposed to otherwise. Instead, this condition will be inflicted without the drop or flip featuring a `Woobly` [animstate](../../Entities/EntityControl/Animations/animstate.md) and it will be removed after most attacks which allows the drop or flip to occur on any subsequent hit|
|9|[Flipped](BattleCondition/Flipped.md)|An ephemeral condition with optional stopping behaviours specific to enemy party members that is only inflicted with a `Flip` [AttackProperty](../Damage%20pipeline/AttackProperty.md) to an enemy party member with a `Flip` [weakness](../Damage%20pipeline/AttackProperty.md). It causes all the enemy party member's `def` to be ignored during damage calculation which is recognised by [TrueDef](../Visual%20rendering/RefreshEnemyHP.md). This process can be hampered by [Topple](BattleCondition/Topple.md)'s defense mechanism if it applies. NOTE: The condition logic has caveats, check the [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) documentation to learn more|
|10|[Shield](BattleCondition/Shield.md)|A condition that encases the actor in a shield by controlling its battleentity.`shieldenabled` which allows it to bypass damage calculation and not loose its `plating`|
|11|[Taunted](BattleCondition/Taunted.md)|A condition that prevents a player party member from selecting the `Skill`, `Item` and `Relay` vine menu [Actions](../Player%20UI/Actions.md) and prevents other player party member from relaying to them|
|12|[Sturdy](BattleCondition/Sturdy.md)|A condition that prevents other conditions from being inflicted (even through using an [item](../../Enums%20and%20IDs/Items.md)) while also getting a -3 damage effect applied in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) at the expense that the actor cannot be relayed to|
|13|[GradualHP](BattleCondition/GradualHP.md)|A condition that increases the actor's `hp` by 2 (clamped from 0 to `maxhp`) every main turn while active|
|14|[GradualTP](BattleCondition/GradualTP.md)|A condition that increases instance.`tp` by 2 (clamped from 0 to instance.`maxtp`) every main turn while active|
|15|[Eaten](BattleCondition/Eaten.md)|A special stopping condition specific to the `Pitcher` [enemy](../../Enums%20and%20IDs/Enemies.md) that allows through unorthodox methods to treat a player party member as if they were dead alongside having their `eatenby` not null|
|16|[EventStop](BattleCondition/EventStop.md)|A stop condition specifically made for [event](../../Enums%20and%20IDs/Events.md) 182 (trying to exit the room after approaching the tank in the last room of Upper Snakemouth) that simply disables a party member until it is manually removed (it is not meant to expire naturally)|
|17|[Fire](BattleCondition/Fire.md)|A condition that deals non lethal damage (more than [Poison](BattleCondition/Poison.md)) every main turn while it is active that may be inflicted in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) if property is `Fire` (without a resistance field) featuring a `Prefabs/Particles/Flame` as a `firepart`|
|18|[Inked](BattleCondition/Inked.md)|A condition that prevents a player party member from using a [skill](../../Enums%20and%20IDs/Skills.md) that may be inflicted in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) if property is `Ink` or `InkOnBlock` (without a resistance field) featuring a `Prefabs/Particles/InkDrip` as a `firepart`|
|19|[Sticky](BattleCondition/Sticky.md)|A condition that prevents a player party member from using an [item](../../Enums%20and%20IDs/Items.md) that may be inflicted in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) if property is `Sticky` (without a resistance field) featuring a `Prefabs/Particles/StickyDrip` as a `firepart`|
|20|[Reflection](BattleCondition/Reflection.md)|A condition that causes a player party member to sustain less damage from [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) with the reduction amount being the amount of `Reflection` [medal](../../Enums%20and%20IDs/Medal.md) equipped. It always expire on the next main turn, but the turn counter is used to visually show the amount of the damage reduction|
