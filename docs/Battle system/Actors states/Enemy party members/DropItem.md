# DropItem
This method will make an enemy party member drop its `helditem` and reset `holditem` accordingly.

```cs
private void DropItem(ref MainManager.BattleData target, bool additem)
```

## Parameters

- `target`: The enemy party member to have its `helditem` dropped. NOTE: it is invalid to send a player party member
- `additem`: Whether or not the `holditem` should be added to `items[0]` (standard items inventory) before its drop

## Procedure

- The `Fall` sound is played
- If additem is true, the enemy party member has a `holditem` and the amount of `items[0]` (standard items) is less than instance.`maxitems`, the `holditem` is added to `items[0]`
- The enemy party member's `holditem` is set to -1
- ItemDrop is called with the enemy party member's `helditem` which does the following:
    - Root `helditem`
    - Add a RigidBody to `helditem` with a velocity of RandomItemBounce(5.0, 12.5)
    - `helditem` gets destroyed in 1.0 second
- `helditem` is set to null
