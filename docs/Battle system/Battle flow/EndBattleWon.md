# EndBattleWon
This method is a wrapper around [AddExperience](Terminal%20coroutines/AddExperience.md) that is mainly used for [EventDialogues](EventDialogue.md) to end the battle in a win.

```cs
private void EndBattleWon(bool addexp, int[] skipids)
```

## Parameters

- `addexp`: Whether the `exp` of every remanining enemy party members should be added to `expreward`
- `skipids`: If `addexp` is true, the list of enemy party members index to not have their `exp` added to `expreward`. This is ignored if `addexp` is false

## Procedure

- If addexp is true, all enemy party members's `exp` except the ones whose index is in `skipids` are added to `expreward` followed by a [RefreshEXP](../Visual%20rendering/RefreshEXP.md) call
- `enemydata` is reset to a new empty array
- `cancelupdate` is set to true changing to a [terminal flow](Update.md#terminal-flow) (but it will already be changed to it anyway right after)
- An [AddExperience](Terminal%20coroutines/AddExperience.md) call starts which also makes sure to be in a [terminal flow](Update.md#terminal-flow)
