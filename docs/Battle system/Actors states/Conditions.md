# Conditions
A condition is a temporary effect applied for a limited amount of turns on an actor. They are tracked in the actor's `condition` array.

A `condition` element contains an int\[\] of 2 elements:

- 0: The `BattleCondition` (in int form) representing the kind of effects this condition involves
- 1: The amount of turns left for the effects to apply. 0 and below values are technically allowed, but frequently causes the whole condition to be removed and they are not considered to be present in the actor

Some conditions are tested for resistance before being added to `condition`. This usually involves the actor's resistance compared to a random number with immunity starting at a resistance of 100 and a certainty of infliction being 0 with everything in between being (100 - resistance) / 100 chance to inflict. The presence of this test, its methodology and its accuracy (not all have the correct odds) depends on the specific method used to add the condition.

Conditions are used heavily throughout the battle system and even come with a set of utility methods to inflict / amend one, check the presence of one and remove one. Here are links to learn more about these methods:

- MainManager.[SetCondition](Conditions%20methods/SetCondition.md)
- [StatusEffect](Conditions%20methods/StatusEffect.md)
- MainManager.[HasCondition](Conditions%20methods/HasCondition.md)
- MainManager.[RemoveCondition](Conditions%20methods/RemoveCondition.md)
