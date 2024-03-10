# Uncontrolled flow
This is one of the flow of [Update](../Update.md) which happens when Update delegates most controls to an action coroutine.

This flow happens if `action` or `inevent` is true which means that an action coroutine or an EventDialogue took control of the battle flow. This means that Update relegated its control and it features a very reduced logic compared to a [controlled flow](Controlled%20flow.md). It is meant to be entered temporarily where control will eventually be given back to a controlled flow with the exception of [terminal](Terminal%20flow.md) cases which ends or retries the battle.

The main feature of this flow is less UI being shown as they are mostly disabled.

Here are all the logic included in this flow.

## RefreshEXP
If the `oldexp` value doesn't match the current `expreward`, [RefreshEXP](../../Visual%20rendering/RefreshEXP.md) is called. This method manages the visual rendering of the currently cumulated EXP and the different orbs that composes it. `expreward` is the actual EXP cumulated so far while `oldexp` is the last value observed since the last RefreshEXP. It will also create `expholder` if it wasn't created already.

## HP UI refresh
[RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called every 3 frames. It should be noted that in this flow, this will always results in the `hpbar` of the enemies to be disabled. This means this refresh is less useful here, but it's more useful in a controlled flow.

## UpdateSwitchIcon
UpdateSwitchIcon is called. It should be noted that in this flow, this will always results in the `switchicon` to be disabled. This means this refresh is less useful here, but it's more useful in a controlled flow.

## Blocking updates
This section happens only when all the following are fufilled:

- `enemy` is true (we are in the player phase)
- `inevent` is false (no [EventDialogue](../EventDialogue.md) is in progress)
- The [message](../../../SetText/Notable%20states.md#message) lock is released (covers cases like [Tattle](../Action%20coroutines/Tattle.md))

[GetBlock](../GetBlock.md) is called here which updates `blockcooldown` and `commandsuccess` according to the current blocking state. Consult the GetBlock documentation to learn more about how the blocking system works.

This is the only logic that is exclusive to an uncontrolled flow.
