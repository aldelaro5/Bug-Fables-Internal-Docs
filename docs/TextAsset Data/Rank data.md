# Rank data

The game contains a TextAsset located at `Ressources/data/LevelData` from the root of the assets tree that contains data about special bonuses gained for specific ranks reached after a rank up. It is loaded each time in LevelUpMessage (which also applies the bonus on top of presenting the message) and [Event](../SetText/Individual%20commands/Event.md) 208 (Talking to Eetl in RUIGEE) to gather the bonus information if one exist. If no bonus exists for the given new rank, no extra bonus is applied on top of the standard one (chosen in battle or 1 MP under RUIGEE).

The asset contains a list of bonuses to apply given a rank reached, one per line. Each lines contains fields:

|Name|Type|Description|
|----|----|-----------|
|Rank|int|The rank to reach for this bonus to be given|
|Bonus type|int|The type of the bonus to apply|
|First parameter|int|The first parameter of the bonus to apply|
|Second parameter|int|The second parameter of the bonus to apply|
|Third parameter|int|The third parameter of the bonus to apply|

The available bonus types are the followings along with their parameters meaning:

|Id|Description|Parameters|
|--:|-----------|----------|
|0|Grants a [skill](../Enums%20and%20IDs/Skills.md) to a party member|First: [AnimID](../Enums%20and%20IDs/AnimIDs.md) of the recipient of the skill|
|--|--|Second: Skill id to grant|
|--|--|Third: NOT USED|
|1|Grant a stat bonus to a party member|First: [AnimID](../Enums%20and%20IDs/AnimIDs.md) of the recipient of the stat bonus|
|--|--|Second: Stat to increase (0 = Attack, 1 = Defense, 2 = HP)|
|--|--|Third: Amount to increase the stat|
|2|Grants a stat bonus to the whole party|First: The stat to increase (0 = TP, MP otherwise)|
|--|--|Second: Amount to increase the stat|
|--|--|Third: NOT USED|
|3|Grant an inventory capacity increase|First: Amount to increase the capacity|
|--|--|Second: NOT USED|
|--|--|Third: NOT USED|

## About [skills](../Enums%20and%20IDs/Skills.md) rank up data
The bonus type 0 doesn't actually grant anything. The actual rank check is done in [RefreshSkills](../Battle%20system/RefreshSkills.md), but the data is still used to present the message informing that a skill was available. That however means there is 2 sources of truths about this: the data tells what the game needs to present the message of getting the skill while RefreshData tells if the skill is actually available with the new rank. For the skill availability to function correctly, both information should match. Failure to do so will cause the rank up message to advertise a new skill, but it wouldn't be available or vice versa.

## A few notes about how the data is handled

For bonus type 0 and 1, the [AnimID](../Enums%20and%20IDs/AnimIDs.md) can only be 0 (`Bee`), 1 (`Beetle`) or 2 (`Moth`) because not doing so can lead to undefined behaviors. Specifically, the game will not be able to get the correct member's name as it is expected to be at menutext 46, 47 and 48 respectively. Any other will resolve to the wrong menutext as it will go past them.

Additionally, the game has a provision that disallow showing the rank up upgrade message if no member with the [AnimID](../Enums%20and%20IDs/AnimIDs.md) exists in the party. The bonus will be applied regardless, but the [SetText](../SetText/SetText.md) command will not be done.

Bonus type 1 and 2 are ignored under RUIGEE.
