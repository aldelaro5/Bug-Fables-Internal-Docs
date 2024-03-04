# RevivePlayer
This method properly revive a player party member.

```cs
private void RevivePlayer(int id, int hp, bool showcounter)
```

## Parameters

- `id`: The `playerdata` index to revive
- `hp`: The `hp` to set the player party member. If it's negative, the `hp` will not be changed. NOTE: this should only be done when something else is guaranteed to heal the player party member as failure to do so will lead the `hp` left dangling at 0 while the actor isn't fully dead
- `showcounter`: Whether to call [Heal](../Heal.md) or not as part of the `hp` increase

## Procedure

- battleentity.`dead` is set to false
- battleentity.`iskill` is set to false
- battleentity has [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md) called on it which unlocks its `rigid`
- battleentity.`shadow` is enabled
- battleentity.`nocondition` is set to false
- The `playerdata`'s `turnssincedeath` is set to 0
- If the player party member is included in `deadmembers`:
    - The `playerdata`'s `cantmove` is set to 0 (one action available) unless `enemy` is true (we are in the [enemy phase](../../Battle%20flow/Update.md#enemies-phase) or later) where it's set to 1 instead (one actor turn is needed for an action to be available)
    - [ClearStatus](../Conditions%20methods/ClearStatus.md) is called on the player party member
    - The `playerdata`'s `moreturnnextturn` is set to 0
    - The `playerdata`'s `tired` is set to 0
    - battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 13 (`BattleIdle`)
- If the hp isn't negative:
    - If showcounter is false, the player party member's `hp` is increased by hp clamped from 0 to its `maxhp`
    - Otherwise, [Heal](../Heal.md) is called on the player party member with the hp amount to heal
- If battleentity.`deathcoroutine` is in progress, it is stopped
- battleentity.`deathcoroutine` is set to null
- A ReviveFix coroutine is started with the id sent which will do the following:
    - 0.5 seconds are yielded
    - If battleentity.`deathcoroutine` is in progress, it is stopped
    - battleentity.`deathcoroutine` is set to null
    - battleentity.`dead` is set to false
    - battleentity.`iskill` is set to false
    - battleentity has [LockRigid(false)](../../../Entities/EntityControl/EntityControl%20Methods.md) called on it which unlocks its `rigid`
    - battleentity.`shadow` is enabled
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called

