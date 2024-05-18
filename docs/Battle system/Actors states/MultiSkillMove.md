# MultiSkillMove
This is a method that is exclusively meant to be used in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) where a player action involving multiple player party members is used. It is called typically at the end of such player action's logic.

It is meant to work alongside the normal actor turn consumption logic performed in [post-action](../Battle%20flow/Action%20coroutines/DoAction.md#post-action) by consuming the actor turns of the all of the involved player party memebrs (except the attacker which is advanced normally by post-action) and some logic associated.

It is hardcoded to apply to the following actions (because these actions will not have the attacker's `tired` incremented as this method will do it for all player party members involved):

- 5 (`BeeFly`)
- 26 (`IceBeemerang`)
- 27 (`IceDrill`)
- 31 (`IceSphere`)

```cs
private void MultiSkillMove(int[] ids)
```

## Parameters

- `ids`: The `playerdata` indexes of all the involved player party members that participated in the player action

## Procedure

Every player party members involved in ids will have the following happen:

- `cantmove` incremented
- `charge` set to 0
- `tired` incremented

Finally, the `currentturn` player party member gets `cantmove` decremented. This undoes the increment that just happened because their actor turn will get advanced normally by post-action which avoids to advance it twice.
