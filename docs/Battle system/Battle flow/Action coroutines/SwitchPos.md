# SwitchPos
This action coroutine alloews to swap 2 player party member's position with each other.

```cs
private IEnumerator SwitchPos(int called, int targeted)
```

> NOTE: The process of switching is complex and involves [battle `playerdata` addressing documentation](../../playerdata%20addressing.md#methods-of-addressing-durring-battle). It is recommended to check this documentation to fully understand how a party swap works.

## Parameters

- `called`: The `playerdats` index that is requesting the swap
- `targeted`: The `playerdata` index that was targetted for swapping by the `called`'s player party member's action

## Procedure

- The `Switch` sound is played
- `action` is set to true changing to an [uncontrolled flow](../Update.md#uncontrolled-flow)
- `currentturn` is set to -1 which unselects the current player party member
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
- The called and targeted player party members has their battleentity.`spin` set to (0.0, -20.0, 0.0)
- 0.33 seconds are yielded
- The `partypointer` indexes that maps to the called and targeted `playerdata` indexes are found and their value are set to called and targeted respectively (essentially, this converts called and targeted from `playerdata` indexes to `partypointer` indexes)
- Both `partypointer` elements whose indexes matches the ones found are swapped. This essentially completes the first party of the swap
- 0.1 seconds are yielded
- For each player party members, the second part of the swap is performed:
    - battleentity.`overridejump` is set to true
    - battleentity.`spin` is zeroed out
    - `pointer` is set to the matching `partypointer`. This gives a way to do the same function than `partypointer`, but using `playerdata` indexes. In practice, this feature is UNUSED because the field is only written to
    - `playerdata[partypointer[j]]`.battleentity position is set to the matching `partypos`. This can be thought as the player party member who had this formation position in the player party is moved to be physically at the matching position of that formation position
    - If battleentity.`deathcoroutine` is in progress, it is stopped and set to null followed by battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) being set to 18 (`KO`)
- [CancelList](../../Player%20UI/CancelList.md) is called
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- A frame is yielded
- `action` is set to false changing to a [controlled flow](../Update.md#controlled-flow)
- A frame is yielded (this frame will process in a controlled flow, but the rest is just visual updates so it's safe)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
