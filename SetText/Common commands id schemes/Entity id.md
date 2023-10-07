# Entity ID

An entity id is an int value sent to the GetEntity method in MainManager which is used often when parsing [SetText](../SetText.md) parameters. This page explains how this format works. Please note it only concerns the int form as most [Commands](Commands.md) supports this form, but not all of them supports other designator such as [define](Individual%20commands/Define.md), refer to each [Commands](Commands.md) documentation to learn more.

## When not in battle

An entity index is one of the following when not in battle:

* \>= 1000: Refers to a temporary non party follower whose index is the index - 1000. If no such follower exist at that index, an exception will be thrown.
* -1: Refers to the first player's entity if it exist, otherwise, this is null.
* -2: Refers to the second player's entity if it exist, otherwise,  this is null.
* -3: Refers to the third player's entity if it exist, otherwise,  this is null.
* -4: Refers to Vi's player entity if it exist, otherwise,  this is null.
* -5: Refers to Kabbu's player entity if it exist, otherwise,  this is null.
* -6: Refers to Leif's player entity if it exist, otherwise,  this is null.
* Any negative value below -6: This is null.
* Other positive values: Refers to the entity in the current map at the index corresponding to the value. If no such entity exists at the index, this is null.

## When in battle

When in battle, the entity index is one of the following:

* If it is between 0 and the amount of player entities in the party - 1, it refers to the corresponding battle player entity at the index of playerdata (this will throw an exception if no player data exists at that index).
* If it is equal or above the amount of player entities in the party, it refers to the enemy battle entity at index - amount of player entities in the party (this will throw an exception if no such enemy data exists at that index).
* Otherwise, the index refers to a battle's extra entity at the absolute value of the index + 1. This is null if no extra battle entities exists (this will throw an exception if there are extra entities, but none exist at the index).
