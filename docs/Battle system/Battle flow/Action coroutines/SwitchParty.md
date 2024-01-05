# SwitchParty
This action coroutine alloews to rotate the party once. It is possible to do it without a fancy visual animation.

```cs
private IEnumerator SwitchParty(bool fast)
```

> NOTE: The process of switching is complex and involves [battle `playerdata` addressing documentation](../../playerdata%20addressing.md#methods-of-addressing-durring-battle). It is recommended to check this documentation to fully understand how a party rotation works.

## Parameters

- `fast`: Whether to include vidual animations and waiting.

## Procedure

- `action` is set to true changing to an [uncontrolled flow](../Update.md#uncontrolled-flow)
- `currentturn` is set to -1 which unselects the current player party member
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
- If fast is false
    - All player party members has their battleentity.`spin` set to (0.0, -20.0, 0.0)
    - 0.33 seconds are yielded
- MainManager.SwitchParty is called with the battle version which rotates the `partypointer` elements
- 0.1 seconds are yielded
- For each player party members, the second part of the rotation is performed:
    - battleentity.`overridejump` is set to true
    - battleentity.`spin` is zeroed out
    - `pointer` is set to the matching `partypointer`. This gives a way to do the same function than `partypointer`, but using `playerdata` indexes. In practice, this feature is UNUSED because the field is only written to
    - `playerdata[partypointer[j]]`.battleentity position is set to the matching `partypos`. This can be thought as the player party member who had this formation position in the player party is moved to be physically at the matching position of that formation position
    - If battleentity.`deathcoroutine` is in progress, it is stopped and set to null followed by battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) being set to 18 (`KO`)
- 0.7 seconds are yielded if fast is false
- [CancelList](../../Player%20UI/CancelList.md) is called
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- A frame is yielded
- `action` is set to false changing to a [controlled flow](../Update.md#controlled-flow)
- A frame is yielded (this frame will process in a controlled flow, but the rest is just visual updates so it's safe)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
