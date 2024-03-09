# GotoSelect
If the confirmation handling needs to move to target selection (which happens from [SetItem](SetItem.md)), a method called GotoSelect is involved receiving a boolean called overridemax which prepares the state for [GetChoiceInput](GetChoiceInput.md) to handle this.

```cs
private void GotoSelect(bool overridemax)
```

Here is the logic of that method depending on the [itemarea](../Player%20UI/AttackArea.md) (nothing happens if it's not among them):

## `SingleAlly`

- `maxoptions` is set to the length of `playerdata`
- [currentaction](Pick.md) is set to `SelectPlayer`

## `SingleEnemy`

- If overridemax is false, `maxoptions` is set to the length of `availabletargets`
- [currentaction](Pick.md) is set to `SelectEnemy`

## `AllEnemies`

- `option` is set to 0
- `maxoptions` is set to 1
- [currentaction](Pick.md) is set to `SelectEnemy`

## `AllParty`

- `option` is set to `currentturn`
- `maxoptions` is set to 1
- [currentaction](Pick.md) is set to `SelectPlayer`

## `All`

- `option` is set to 0
- `maxoptions` is set to 1
- [currentaction](Pick.md) is set to `SelectEnemy`
